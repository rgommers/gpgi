project('gpgi', 'cython', meson_version: '>=1.5.0')

py = import('python').find_installation(pure: false)

min_numpy_version = '1.25.0'  # keep in sync with pyproject.toml
cy = meson.get_compiler('cython')
cython = find_program(cy.cmd_array()[0])

numpy_nodepr_api = ['-DNPY_NO_DEPRECATED_API=NPY_1_25_API_VERSION']

# copied from PyWavelets
# https://github.com/PyWavelets/pywt/blob/becef5486174c77727ea2a60e6392744b2cc1d4a/pywt/_extensions/meson.build#L13
# Uses the `numpy-config` executable (or a user's numpy.pc pkg-config file),
# will work for numpy>=2.0.0b1 and meson>=1.4.0
_numpy_dep = dependency('numpy', required: false)
if _numpy_dep.found()
  np_dep = declare_dependency(dependencies: _numpy_dep, compile_args: numpy_nodepr_api)
else
  # For cross-compilation it is often not possible to run the Python interpreter
  # in order to retrieve numpy's include directory. It can be specified in the
  # cross file instead:
  #   [properties]
  #   numpy-include-dir = /abspath/to/host-pythons/site-packages/numpy/core/include
  #
  # This uses the path as is, and avoids running the interpreter.
  incdir_numpy = meson.get_external_property('numpy-include-dir', 'not-given')
  if incdir_numpy == 'not-given'
    incdir_numpy = run_command(py,
      [
        '-c',
        '''import os
import numpy as np
try:
  incdir = os.path.relpath(np.get_include())
except Exception:
  incdir = np.get_include()
print(incdir)
    '''
      ],
      check: true
    ).stdout().strip()
  endif
  inc_np = include_directories(incdir_numpy)
  np_dep = declare_dependency(include_directories: inc_np, compile_args: numpy_nodepr_api)
endif

py.extension_module(
    '_lib',
    'src/gpgi/_lib.pyx',
    subdir: 'gpgi',
    install: true,
    dependencies : [np_dep],
    c_args: numpy_nodepr_api,
)

# meson doesn't have an equivalent to setuptools.packages.find
# at the time of writing, so, listing every module explicitly here
py.install_sources(
    'src/gpgi/__init__.py',
    'src/gpgi/_boundaries.py',
    'src/gpgi/_data_types.py',
    'src/gpgi/_load.py',
    'src/gpgi/_spatial_data.py',
    'src/gpgi/_typing.py',
    'src/gpgi/typing.py',
    subdir: 'gpgi'
)
