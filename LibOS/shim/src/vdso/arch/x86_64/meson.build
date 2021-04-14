libos_vdso_so_dbg = shared_library('vdso',
    'vdso.c',
    name_prefix: '',
    name_suffix: 'so.dbg',
    include_directories: includes_libos,
    link_args: [
        '-T@0@'.format(files('vdso.lds')),
        '-nostdlib',
        '-shared',
        '--hash-style=both',
        '--build-id',
        '-Bsymbolic',
        '-melf_x86_64',
        '--no-undefined',
        '-zmax-page-size=4096',
        '-zcommon-page-size=4096',
        '--soname=linux-vdso.so.1',
    ],
)

# TODO check_no_reloc

libos_sources += custom_target(
    'vdso.so',
    command: [objcopy, '-S', '@INPUT@', '@OUTPUT@'],
    input: libos_vdso_so_dbg,
    output: 'vdso.so',
)
