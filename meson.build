project(
  'minimap2',
  'c',
  version : '2.17',
  default_options : [
    'buildtype=release',
    'warning_level=3',
    'c_std=c11',
    'b_ndebug=if-release'],
  license : 'MIT',
  meson_version : '>= 0.46.0')

##########
# CFLAGS #
##########

mm2_flags = ['-DHAVE_KALLOC']
c = meson.get_compiler('c')
foreach cflag: [
  '-Wc++-compat',
  '-Wno-unused-parameter',
  '-Wno-unused-variable',
  '-Wno-sign-compare',
  '-Wno-unused-but-set-variable']
    if c.has_argument(cflag)
      mm2_flags += cflag
    endif
endforeach

################
# dependencies #
################

# threads
mm2_thread_dep = dependency('threads', required : true)

# zlib
mm2_zlib_dep = dependency('zlib', required : true, fallback : ['zlib', 'zlib_dep'])

# math
mm2_math_dep = c.find_library('m')

mm2_deps = [mm2_thread_dep, mm2_zlib_dep, mm2_math_dep]

###########
# headers #
###########

if not meson.is_subproject()
  install_headers(
    files([
      'bseq.h',
      'kalloc.h',
      'ketopt.h',
      'kdq.h',
      'khash.h',
      'kseq.h',
      'ksort.h',
      'ksw2.h',
      'kthread.h',
      'kvec.h',
      'minimap.h',
      'mmpriv.h',
      'sdust.h']),
    subdir : 'minimap2')
endif

mm2_include_directories = include_directories('.')

#############
# libraries #
#############

simd_mod = import('unstable_simd')
mm2_simd_flag = []
mm2_simd_libs = []

mm2_simd_libs += simd_mod.check(
  'll_sse',
  sse2 : files([
    'ksw2_ll_sse.c']),
  include_directories : mm2_include_directories,
  c_args : mm2_flags,
  compiler : c)[0]

if get_option('sse41')
  # SSE 4.1
  mm2_simd_flag += ['-DKSW_CPU_DISPATCH']

  mm2_simd_libs += simd_mod.check(
    'ksw2_sse41',
    sse41 : files([
      'ksw2_extd2_sse.c',
      'ksw2_exts2_sse.c',
      'ksw2_extz2_sse.c']),
    include_directories : mm2_include_directories,
    c_args : [mm2_flags, mm2_simd_flag],
    compiler : c)[0]

  mm2_simd_libs += simd_mod.check(
    'ksw2_dispatch',
    sse41 : files([
      'ksw2_dispatch.c']),
    include_directories : mm2_include_directories,
    c_args : [mm2_flags, mm2_simd_flag],
    compiler : c)[0]

  mm2_simd_flag += ['-DKSW_SSE2_ONLY']
endif

# SSE2
mm2_simd_libs += simd_mod.check(
  'ksw2_sse2',
  sse2 : files([
    'ksw2_extd2_sse.c',
    'ksw2_exts2_sse.c',
    'ksw2_extz2_sse.c']),
  include_directories : mm2_include_directories,
  c_args : [mm2_flags, mm2_simd_flag],
  compiler : c)[0]

# this is a crutch to unify all compiled objects into
# one library, in order to avoid tons of libraries
# showing up in the pkg-config file
# https://github.com/mesonbuild/meson/issues/2478#issuecomment-351132872
mm2_simd_all_objects = []
foreach i : mm2_simd_libs
  mm2_simd_all_objects += i.extract_all_objects()
endforeach

# install library if
# - either running as a proper project
# - or using shared libraries
mm2_lib_install = (not meson.is_subproject()) or (get_option('default_library') == 'shared')

mm2_cpp_sources = files([
  'align.c',
  'bseq.c',
  'chain.c',
  'esterr.c',
  'format.c',
  'hit.c',
  'index.c',
  'kalloc.c',
  'kthread.c',
  'map.c',
  'misc.c',
  'options.c',
  'pe.c',
  'sdust.c',
  'sketch.c',
  'splitidx.c'])

mm2_lib = library(
  'minimap2',
  mm2_cpp_sources,
  objects : mm2_simd_all_objects,
  install : mm2_lib_install,
  # use boost SONAME practice:
  #   cause ld startup issues before
  #   you even have the chance of running
  #   into ABI issues.
  soversion : meson.project_version(),
  version : meson.project_version(),
  dependencies : mm2_deps,
  include_directories : mm2_include_directories,
  c_args : mm2_flags)

###############
# executables #
###############

# minimap2
if not meson.is_subproject()
  mm2_main = executable(
    'minimap2', files([
      'main.c']),
    install : true,
    dependencies : mm2_deps,
    include_directories : mm2_include_directories,
    link_with : mm2_lib,
    c_args : mm2_flags)
endif

###################
# dependency info #
###################

if not meson.is_subproject()
  import('pkgconfig').generate(
    libraries : mm2_lib,
    version : meson.project_version(),
    name : 'minimap2',
    filebase : 'minimap2',
    subdirs : 'minimap2',
    description : 'A versatile pairwise aligner for genomic and spliced nucleotide sequences')
endif

mm2_dep = declare_dependency(
  include_directories : mm2_include_directories,
  link_with : mm2_lib,
  dependencies : mm2_deps,
  version : meson.project_version())
