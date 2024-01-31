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
