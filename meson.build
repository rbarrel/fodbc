project(
  'fodbc',
  'c',
  'fortran',
  version: '1.0.0',
  license: 'BSD 3-Clause',
  default_options: [
    'buildtype=debugoptimized',
    'default_library=both',
  ],
)
install = not (meson.is_subproject() and get_option('default_library') == 'static')

odbc_dep = dependency('odbc', required: true)

srcs = []
subdir('src')

fodbc_lib = library(
  meson.project_name(),
  sources: srcs,
  install: install,
  dependencies: odbc_dep,
)

pkg = import('pkgconfig')
pkg.generate(
  fodbc_lib,
  description: 'Fortran interface to ODBC.',
)

# Export fodbc as a Dependency for Other Projects
fodbc_inc = fodbc_lib.private_dir_include()
fodbc_dep = declare_dependency(
  link_with: fodbc_lib,
  include_directories: fodbc_inc,
)
