# fodbc

An ODBC connector for the Fortran programming language.
Forked from: https://sourceforge.net/p/flibs/svncode/HEAD/tree/trunk/src/odbc/

The version available upsteam didn't compile, so this repository contains the changes necessary to get the source code to compile independently.

# Installation

Requirements:

Description | Suggested Package
--- | ---
A C Compiler | [GNU GCC (12.2.0)](https://gcc.gnu.org/)
A Fortran Compiler  | [GNU Fortran (12.2.0)](https://gcc.gnu.org/fortran/)
SQL Headers | [unixODBC (2.3.11)](https://www.unixodbc.org/)
A Build System | [The Meson Build System (1.3.0)](https://mesonbuild.com/)
A Backend for Build System | [Ninja Build (1.11.1)](https://ninja-build.org/)

# License

Like Arjen Markus' original, fodbc is licensed under the BSD 3-Clause license. For the full license text, see the LICENSE file.

# Usage

## Name

flibs/odbc - Interface to ODBC

## Table Of Contents

  * Table Of Contents
  * Synopsis
  * Description
  * DATA TYPES
  * ROUTINES
  * ODBC-SPECIFIC ROUTINES
  * EXAMPLE
  * LIMITATIONS
  * IMPLEMENTATION NOTES
  * PLATFORM ISSUES
  * Copyright

## Synopsis

  * **type(ODBC_DATABASE)**
  * **type(ODBC_STATEMENT)**
  * **type(ODBC_COLUMN)**
  * **call odbc_column_props( column, name, type, length )**
  * **call odbc_column_query( column, name, type, length, function )**
  * **call odbc_set_column( column, value )**
  * **call odbc_get_column( column, value )**
  * **call odbc_open( filename_or_data_set_name, driver, db )**
  * **call odbc_connect( connection_string, db )**
  * **call odbc_close( db )**
  * **err = odbc_error( db )**
  * **call odbc_set_blob_support( db, blob_type )**
  * **errmsg = odbc_errmsg( db_or_stmt )**
  * **errmsg = odbc_errmsg_print( db_or_stmt, lun )**
  * **call odbc_do( db, command )**
  * **call odbc_begin( db )**
  * **call odbc_commit( db )**
  * **call odbc_rollback( db )**
  * **call odbc_create_table( db )**
  * **call odbc_delete_table( db )**
  * **call odbc_prepare_select( db, tablename, columns, stmt, extra_clause )**
  * **call odbc_prepare( db, command, stmt, columns )**
  * **call odbc_step( stmt, completion )**
  * **call odbc_reset( stmt )**
  * **call odbc_finalize( stmt )**
  * **call odbc_next_row( stmt, columns, finished )**
  * **call odbc_insert( db, tablename, columns )**
  * **call odbc_get_table( db, commmand, result, errmsg )**
  * **call odbc_query_table( db, tablename, columns )**
  * **call odbc_get_data_source( next, dsnname, description, success )**
  * **call odbc_get_driver( next, driver, description, success )**
  * **call odbc_get_table_name( db, next, table, description, success )**

## Description

The _ODBC_ module provides a Fortran interface to the Open Database
Connectivity system or ODBC. The interface has been implemented in such a way,
that you can use a high-level interface for common tasks, such as inserting
data into a database and querying the contents, as well as lower-level
functionality, accessible via SQL statements, for instance.

To this end the module defines a set of routines and functions as well as
several derived types to hide the low-level details.

In its current form, it does not provide a full Fortran API to all the
functionality offered by SQLite, but it should be quite useable.

_Note:_ This interface has been modelled after the Fortran SQLite interface in
this same project. Because ODBC is not a database management system in its own
right, but instead an common interface to various database systems, several
additional routines are available, such as odbc_get_driver, that have no
equivalent within the context of SQLite.

_Note:_ While ODBC is intended to provide a generic interface to database
management systems, there are still a number of issues that you should be
aware that depend on the operating system and the specific database management
system.

These issues are documented in PLATFORM ISSUES.

## DATA TYPES

The following derived types are defined:

**type(ODBC_DATABASE)**

    

Variables of this type are used to hold the connection to the database or
databases. They are created by the subroutine _odbc_open_

The contents are valid until the database is closed (via _odbc_close_ ).

**type(ODBC_STATEMENT)**

    

Variables of this type hold _prepared statements_ , the common method for
database management systems to efficiently execute SQL statements.

**type(ODBC_COLUMN)**

    

To provide easy communication with the database, ODBC_COLUMN can hold values
of different types. This means you can use a single routine and variable to
transfer strings, integers or reals to and from the database.

The first two derived types are "opaque", that is they are used only to
communicate between the application and the database library and there is
information of interest to be gotten from them.

The third type is rather crucial to the working of the implementation: By
setting the properties of an ODBC_COLUMN variable you put data into the
database or you can retrieve data from the database. See the example below for
how this works.

There are a number of routines that are meant to make this easier:

**call odbc_column_props( column, name, type, length )**

    

Set the properties of a column

type(ODBC_COLUMN) _column_

    

The variable that holds the information on the column

character(len=*) _filename_

    

Name of the column in the table to which it belongs or will belong

integer _type_

    

Type of the column: one of ODBC_INT, ODBC_REAL, ODBC_DOUBLE, ODBC_CHAR or
ODBC_BINARY (see PLATFORM ISSUES).

integer, optional _length_

    

Length of a character-valued column (defaults to 20 characters) or a BLOB-type
column.

**call odbc_column_query( column, name, type, length, function )**

    

Set the properties of a column when constructing a SELECT query. The
"function" argument, if present, is a string representing an SQL function like
_count_ or _max_.

type(ODBC_COLUMN) _column_

    

The variable that holds the information on the column

character(len=*) _filename_

    

Name of the column in the table to which it belongs or will belong

integer _type_

    

Type of the column: one of ODBC_INT, ODBC_REAL, ODBC_DOUBLE, ODBC_CHAR or
ODBC_BINARY.

integer, optional _length_

    

Length of a character-valued column (defaults to 20 characters)

character(len=*), optional _function_

    

Name of the SQL function to perform on the values.

**call odbc_set_column( column, value )**

    

Set the _value_ of a column

type(ODBC_COLUMN) _column_

    

The variable that holds the information on the column

any type _value_

    

The new value for the column. The type of the value that is passed can be
integer, real, double precision real or character string.

_Note:_ currently there is no conversion from the type of value that is stored
to the type of the actual variable that is passed to the routine. If you ask
for an integer and the column holds a real, then the result is undefined.
Check the type with the value of the flag "type_set". (This is one of the
things that should be improved)

**call odbc_get_column( column, value )**

    

Get the _value_ of a column

type(ODBC_COLUMN) _column_

    

The variable that holds the information on the column

any type _value_

    

The value stored in the column. The type of the value that is passed can be
integer, real, double precision real or character string.

## ROUTINES

The _odbc_ module currently provides the following functions:

**call odbc_open( filename_or_data_set_name, driver, db )**

    

Open a database by _data set name_ or by _file name and driver_ and store the
connection for later use.

character(len=*) _filename_or_data_set_name_

    

The name of the data set (DSN, as known to the ODBC system) or the database
file to be opened.

character(len=*), optional _driver_

    

The name of the driver, currently one of the _parameters_ ODBC_MSACCESS,
ODBC_MSEXCEL, ODBC_SQLITE or ODBC_POSTGRES (see PLATFORM ISSUES). If left out,
the name is supposed to be a data set name (DSN).

type(ODBC_DATABASE) _db_

    

Variable to identify the database connection

**call odbc_connect( connection_string, db )**

    

Open a connection to the database of choice via a full connection string. This
routine is useful if you want specific connection parameters or if the driver
is not directly supported.

character(len=*) _connection_string_

    

The connection string to be used. It must contain all information required
(see the documentation on the particular ODBC driver).

type(ODBC_DATABASE) _db_

    

Variable to identify the database connection

**call odbc_close( db )**

    

Close the database connection. Simply an interface to the corresponding C
function.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

**err = odbc_error( db )**

    

Retrieve whether the previous command resulted in an error or not. Returns
true if so, otherwise false.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

**call odbc_set_blob_support( db, blob_type )**

    

Set the type of support for BLOBs (see PLATFORM ISSUES). Use this if you
intend to use BLOBs (binary large objects).

type(ODBC_DATABASE) _db_or_stmt_

    

Variable identifying the database connection.

integer _blob_type_

    

Identify how the dabase management system supports BLOBs:

  * _ODBC_PLAIN_BLOB_ \- (default) the database system uses the keyword "BLOB" to indicate binary large objects and the ODBC driver simply returns a set of bytes.

  * _ODBC_POSTGRES_HEX_ \- the database system (notably PostgreSQL) uses the keyword "BYTEA" to indicate binary large objects and the ODBC driver returns a hexdecimally encoded string instead of a set of bytes.

**errmsg = odbc_errmsg( db_or_stmt )**

    

Retrieve the last error message as a string of at most 80 characters.

type(ODBC_DATABASE/ODBC_STATEMENT) _db_or_stmt_

    

Variable identifying the database connection or the statement that produced
the error.

**errmsg = odbc_errmsg_print( db_or_stmt, lun )**

    

Print the last error messages to the screen or to a file

type(ODBC_DATABASE/ODBC_STATEMENT) _db_or_stmt_

    

Variable identifying the database connection or the statement that produced
the error.

integer, optional _lun_

    

LU-number of the file to print the messages to. If not present, the messages
are printed to the screen.

**call odbc_do( db, command )**

    

Run a single SQL command

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _command_

    

String holding a complete SQL command

**call odbc_begin( db )**

    

Start a transaction. When the corresponding routine odbc_commit is called, all
changes will be made permanent. Use a transaction to gather lots of changes to
the database - this is much faster than an automatic commission after each
change.

_Note:_ The database driver may or may not support this feature. Consult the
documentation.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

**call odbc_commit( db )**

    

Commit the changes made since the start of a transaction. This makes the
changes permanent.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

**call odbc_rollback( db )**

    

Undo the changes made since the start a transaction. The database will be
restored to the state it was in before the transaction was started.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

**call odbc_create_table( db )**

    

Create a new table, based on the properties of the columns. Convenience
routine that constructs an SQL statement to do the actual job.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _tablename_

    

Name of the table to be created

type(ODBC_COLUMN), dimension(:) _columns_

    

An array of the properties of the columns in the tables (name, type, ...)

character(len=*), optional _primary_

    

Name of the column that acts as the primary key (this gets the "unique"
constraint)

**call odbc_delete_table( db )**

    

Delete an existing table by name. Convenience routine that constructs an SQL
statement to do the actual job.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _tablename_

    

Name of the table to be deleted

**call odbc_prepare_select( db, tablename, columns, stmt, extra_clause )**

    

Prepare a SELECT query. Convenience routine that creates the SQL query and
"compiles" (prepares) it for later actual execution.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _tablename_

    

Name of the table to be queried

type(ODBC_COLUMN), dimension(:) _columns_

    

An array of the properties of the columns to be returned

type(ODBC_STATEMENT) _stmt_

    

A derived type used as a handle to the prepared statement

character(len=*), optional _extra_clause_

    

A string holding an extra clause, such as "SORT BY" or "GROUP BY"

**call odbc_prepare( db, command, stmt, columns )**

    

Prepare a general SQL statement for later actual execution. The statement can
be any SQL statement.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _command_

    

The SQL statement to be prepared

type(ODBC_STATEMENT) _stmt_

    

A derived type used as a handle to the prepared statement

type(ODBC_COLUMN), dimension(:), pointer _columns_

    

An array of the properties of the columns that will be returned by the
statement. The routine returns an allocated array. You must deallocate it
yourself, when it is no longer needed.

**call odbc_step( stmt, completion )**

    

Run the prepared SQL statement for one step. The code in completion will tell
whether it was successful or not. Simply an interface to the equivalent C
routine.

type(ODBC_STATEMENT) _stmt_

    

A derived type used as a handle to the prepared statement

integer _completion_

    

One of the values ODBC_DONE (success), ODBC_MISUSE or ODBC_ERROR

**call odbc_reset( stmt )**

    

Reset the prepared statement so that it can be used again.

type(ODBC_STATEMENT) _stmt_

    

A derived type used as a handle to the prepared statement

**call odbc_finalize( stmt )**

    

Free all resources associated with the prepared statement.

type(ODBC_STATEMENT) _stmt_

    

A derived type used as a handle to the prepared statement

**call odbc_next_row( stmt, columns, finished )**

    

Retrieve the next row of a SELECT query. If the argument "finished" is set to
true, the previous row was the last one.

type(ODBC_STATEMENT) _stmt_

    

A derived type used as a handle to the prepared statement

logical _finished_

    

Set to true if the last row was retrieved.

**call odbc_insert( db, tablename, columns )**

    

Insert a complete new row into the table.

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _tablename_

    

Name of the table into which the row must be inserted

type(ODBC_COLUMN), dimension(:) _columns_

    

An array of values for all columns

**call odbc_get_table( db, commmand, result, errmsg )**

    

Get the result of a query in a single two-dimensional array

_NOT IMPLEMENTED YET_

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _command_

    

The SQL command (query) to executed

character(len=*), dimension(:,:), pointer _result_

    

A two-dimensional array that will be filled with the results of the SQl
command. When done, you will have to deallocate it.

character(len=*) _errmsg_

    

If there is an error, then "result" will not be allocated, and "errmsg" will
contain the information about the error that occurred.

**call odbc_query_table( db, tablename, columns )**

    

Query the structure of the table

type(ODBC_DATABASE) _db_

    

Variable identifying the database connection

character(len=*) _tablename_

    

Name of the table to be inspected

type(ODBC_COLUMN), dimension(:), pointer _columns_

    

An array with the properties of all columns. Deallocate it when you are done.

## ODBC-SPECIFIC ROUTINES

The following routines are specific to ODBC:

**call odbc_get_data_source( next, dsnname, description, success )**

    

Get the first ( _next = .false._ ) or the next ( _next = .true._ ) data set
name.

logical _next_

    

Whether to get the first or the next data set name

character(len=*) _dsnname_

    

Name of the data set

character(len=*) _description_

    

Description of the data set (usually includes the driver)

logical _success_

    

Whether there is a data set name returned or not

**call odbc_get_driver( next, driver, description, success )**

    

Get the first ( _next = .false._ ) or the next ( _next = .true._ ) registered
driver.

logical _next_

    

Whether to get the first or the next driver

character(len=*) _driver_

    

Name of the driver

character(len=*) _description_

    

Description of the driver

logical _success_

    

Whether there is a driver name returned or not

**call odbc_get_table_name( db, next, table, description, success )**

    

Get information on the first ( _next = .false._ ) or the next ( _next =
.true._ ) table in a database.

logical _next_

    

Whether to get the first or the next table name

character(len=*) _driver_

    

Name of the table

character(len=*), dimension(:) _description_

    

Description of the table (at least 5 elements). The fourth element is the type
of table (SYSTEM_TABLE, TABLE or VIEW).

logical _success_

    

Whether there is a driver name returned or not

## EXAMPLE

To illustrate the usage of the library, here is a small example:

  * Store (fictitious) measurements of salinity and temperature from a CSV file in a single table of a new database.

  * To check that it works, retrieve the average salinity and average temperature per station and print them sorted by station name

The first part of the program simply defines the table:

    
    
       allocate( column(4) )
       call odbc_column_props( column(1), name(1), ODBC_CHAR, 10 )
       call odbc_column_props( column(2), name(2), ODBC_CHAR, 10 )
       call odbc_column_props( column(3), name(3), ODBC_REAL )
       call odbc_column_props( column(4), name(4), ODBC_REAL )
       call odbc_create_table( db, 'measurements', column )
    

The second part reads the data file and stores the data in a new row:

    
    
       call odbc_begin( db )
       do
          read( lun, *, iostat=ierr ) station, date, salin, temp
          if ( ierr .ne. 0 ) exit
          call odbc_set_column( column(1), station )
          call odbc_set_column( column(2), date    )
          call odbc_set_column( column(3), salin   )
          call odbc_set_column( column(4), temp    )
          call odbc_insert( db, 'measurements', column )
       enddo
       close( lun )
       call odbc_commit( db )
    

Note that it uses a transaction (via calls to _odbc_begin_ and _odbc_commit_
pair), so that all the inserts can be done in one go. Inserting with
autocommit is much slower, as the database file needs to be flushed very time.

The last part retrieves the data by constructing an SQL query that will
actually look like:

    
    
        select station, avg(salinity), avg(temperature) from measurements
            grouped by station order by station;
    

The routine _odbc_prepare_select_ takes care of the actual construction of the
above SQL query:

    
    
       deallocate( column )
       allocate( column(3) )
       call odbc_column_query( column(1), 'station', ODBC_CHAR )
       call odbc_column_query( column(2), name(3), ODBC_REAL, function='avg' )
       call odbc_column_query( column(3), name(4), ODBC_REAL, function='avg' )
       call odbc_prepare_select( db, 'measurements', column, stmt, &
          'group by station order by station' )
       write( *, '(3a20)' ) 'Station', 'Mean salinity', 'Mean temperature'
       do
          call odbc_next_row( stmt, column, finished )
          if ( finished ) exit
          call odbc_get_column( column(1), station )
          call odbc_get_column( column(2), salin   )
          call odbc_get_column( column(3), temp    )
          write( *, '(a20,2f20.3)' ) station, salin, temp
       enddo
    

The full program looks like this (see also the tests/examples directory of the
Flibs project):

    
    
    ! csvtable.f90 --
    !    Program to read a simple CSV file and put it into a
    !    SQLite database, just to demonstrate how the Fortran
    !    interface works.
    !
    !    To keep it simple:
    !    - The first line contains the names of the four columns
    !    - All lines after that contain the name of the station
    !      the date and the two values.
    !
    !    $Id: fodbc.html,v 1.3 2013-05-13 08:03:15 knystrom Exp $
    !
    program csvtable
       use odbc
       implicit none
       type(ODBC_DATABASE)                      :: db
       type(ODBC_STATEMENT)                     :: stmt
       type(ODBC_COLUMN), dimension(:), pointer :: column
       integer                                    :: lun = 10
       integer                                    :: i
       integer                                    :: ierr
       character(len=40), dimension(4)            :: name
       real                                       :: salin
       real                                       :: temp
       character(len=40)                          :: station
       character(len=40)                          :: date
       logical                                    :: finished
       !
       ! Read the CSV file and feed the data into the database
       !
       open( lun, file = 'somedata.csv' )
       read( lun, * ) name
       call odbc_open( 'somedata.db', db )
       allocate( column(4) )
       call odbc_column_props( column(1), name(1), ODBC_CHAR, 10 )
       call odbc_column_props( column(2), name(2), ODBC_CHAR, 10 )
       call odbc_column_props( column(3), name(3), ODBC_REAL )
       call odbc_column_props( column(4), name(4), ODBC_REAL )
       call odbc_create_table( db, 'measurements', column )
       !
       ! Insert the values into the table. For better performance,
       ! make sure (via begin/commit) that the changes are committed
       ! only once.
       !
       call odbc_begin( db )
       do
          read( lun, *, iostat=ierr ) station, date, salin, temp
          if ( ierr .ne. 0 ) exit
          call odbc_set_column( column(1), station )
          call odbc_set_column( column(2), date    )
          call odbc_set_column( column(3), salin   )
          call odbc_set_column( column(4), temp    )
          call odbc_insert( db, 'measurements', column )
       enddo
       close( lun )
       call odbc_commit( db )
       !
       ! We want a simple report, the mean of salinity and temperature
       ! sorted by the station
       !
       deallocate( column )
       allocate( column(3) )
       call odbc_column_query( column(1), 'station', ODBC_CHAR )
       call odbc_column_query( column(2), name(3), ODBC_REAL, function='avg' )
       call odbc_column_query( column(3), name(4), ODBC_REAL, function='avg' )
       call odbc_prepare_select( db, 'measurements', column, stmt, &
          'group by station order by station' )
       write( *, '(3a20)' ) 'Station', 'Mean salinity', 'Mean temperature'
       do
          call odbc_next_row( stmt, column, finished )
          if ( finished ) exit
          call odbc_get_column( column(1), station )
          call odbc_get_column( column(2), salin   )
          call odbc_get_column( column(3), temp    )
          write( *, '(a20,2f20.3)' ) station, salin, temp
       enddo
       call odbc_close( db )
    end program
    

## LIMITATIONS

The module is not complete yet:

  * There is no support for blobs or for character strings of arbitrary length. In fact the maximum string length is limited to 80 characters.

  * There is no support for NULL values or for DATE values.

  * The ODBC API is not completely covered, though the subset should be useful for many applications.

  * There are no makefiles that can help build the library yet. See the implementation notes below.

## IMPLEMENTATION NOTES

While the module is fairly straightforward Fortran 95 code, building a library
out of it may not be straightforward due to the intricacies of C-Fortran
interfacing.

This section aims to give a few guidelines:

  * The C code contains all the platform-dependent code, so that the Fortran code could remain clean.

  * To support more than one platform, the C code contains several macros:

    * FTNCALL - the calling convention for Fortran routines (important on Windows). It is automatically set to ___stdcall_ when the macro "WIN32" has been defined (by the compiler or by specifying it on the command-line).

    * INBETWEEN - this macro controls whether the hidden arguments for passing the string length are put inbetween the arguments (if it is defined) or appended to the end (if it is not defined). Under Windows the Compaq Visual Fortran compiler used to use the first method, so this is automatically set. For other platforms the second method is more usual.

The naming convention (additional underscore, all capitals or all lowercase)
has been handled in a simple-minded fashion. This should be improved too.

The library has been designed with 64-bits platforms in mind: it should run on
these platforms without any difficulties.

## PLATFORM ISSUES

  * The library has been tested on Linux, using the _PostgreSQL database_ system with the _psqlODBC_ driver.

It is unclear what the proper connection string should be, so that the type
ODBC_POSTGRES for the routine _odbc_open_ does not work yet. (PostgreSQL has a
client/server architecture and can communicate over TCP/IP, so that more
information may have to be specified than for file-based systems like SQLite
and MS Access. Use the routine _odbc_connect_ directly, so that you can pass a
complete connection string.)

  * As of version 1.1 the library supports so-called binary large objects (column type: ODBC_BINARY). Not all database systems support them and they are actually an extension to the SQL language that underlies the communication to and from the database system. For this reason it may be necessary to use the routine _odbc_set_blob_type_ to identify the database-specific method used for BLOBs. (It does not seem possible to identify this automatically.)
