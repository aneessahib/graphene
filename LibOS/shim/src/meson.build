# don't put this in include_directories, we need this for assembly-only custom_target
includes_libos = []
foreach i : [
    '../include',
    '../include/arch/@0@'.format(build_machine.cpu_family()),
    '../../../Pal/include',
    '../../../Pal/include/arch/@0@'.format(build_machine.cpu_family()),
    '../../../Pal/include/arch/@0@/Linux'.format(build_machine.cpu_family()),
    '../../../Pal/include/elf',
    '../../../Pal/include/lib',
    '../../../Pal/include/pal',
]
    includes_libos += join_paths(meson.current_source_dir(), i)
endforeach

includes_libos_i = []
foreach i : includes_libos
    includes_libos_i += '-I@0@'.format(i)
endforeach

cflags_libos = [
    '-fPIC',
    '-fmerge-all-constants',
    '-fno-builtin',

    '-Werror=implicit-function-declaration',
    '-Wextra',
    '-Winline',
    '-Wno-inline',
    '-Wstrict-prototypes',
    '-Wwrite-strings',

    '-DIN_SHIM',
]

foreach i : [
    '-Wtrampolines',
    '-Wnull-dereference',

    # Some of the code uses alignof on expressions, which is a GNU extension.
    # Silence Clang - it complains but does support it.
    '-Wno-gnu-alignof-expression',

    # TODO: This is due to use of packed structs in IPC code, which triggers "taking address of
    # a packed member" warning in Clang and newer GCC. That code needs to be rewritten.
    '-Wno-address-of-packed-member',
]
    if meson.get_compiler('c').has_argument(i)
        cflags_libos += i
    endif
endforeach

# use TLS-based stack protector of GCC (we rely on the fact that LibOS reuses the same TCB as its
# underlying PAL which must have a canary in its PAL_TCB at offset 0x8, so no additional enabling
# is required in the LibOS code);
# not all compilers support mstack-protector-guard, so use stack protector only if supported
cflags_libos += '-fno-stack-protector'
cflags_libos_stack_protector = [
    '-fstack-protector-strong',
    '-mstack-protector-guard=tls',
    '-mstack-protector-guard-reg=%gs',
    '-mstack-protector-guard-offset=8',
]
if meson.get_compiler('c').has_multi_arguments(cflags_libos_stack_protector)
    cflags_libos += cflags_libos_stack_protector
endif

#LDFLAGS-libsysdb.so += --version-script shim.map -T arch/$(ARCH)/shim.lds --eh-frame-hdr
#libsysdb.so: $(objs) $(filter %.map %.lds,$(LDFLAGS-$@)) \
#	     $(graphene_lib) $(pal_lib) shim.map arch/$(ARCH)/shim.lds
#	$(call cmd,ld_so_o)
# cmd_ld_so_o = ... $(filter-out pal-symbols %.map %.map.template %.lds,$^) -soname $(notdir $@)



libos_generated_offsets_s = custom_target('generated-offsets.s',
    command: [
        meson.get_compiler('c').cmd_array(),
        cflags_libos, includes_libos_i, '-S',
        '@INPUT',
        '-o', '@OUTPUT@'
    ],
    output: 'generated-offsets.s',
    input: 'generated-offsets.c',
)

libos_asm_offsets_h = custom_target(
    command: [asm_offsets_prog, '@INPUT@', '@OUTPUT@'],
    input: libos_generated_offsets_s,
    output: 'asm-offets.h',
)

libos_sources = files(
    libos_asm_offsets_h,

	'shim_async.c',
	'shim_checkpoint.c',
	'shim_debug.c',
	'shim_init.c',
	'shim_malloc.c',
	'shim_object.c',
	'shim_parser.c',
	'shim_rtld.c',
	'shim_syscalls.c',
)

subdir('arch')
subdir('bookkeep')
subdir('fs')
subdir('ipc')
subdir('sys')
subdir('vdso')

shim_map = files('shim.map')[0]

shared_library('sysdb',
    libos_sources,

    link_args: [
        '-Wl,-shared',
        '-Wl,-nostdlib',
        '-Wl,-zcombreloc',
        '-Wl,-zrelro',
        '-Wl,-znow',
        '-Wl,-zdefs',
        '-Wl,--version-script=@0@'.format(shim_map),
        '-Wl,-T@0@'.format(shim_lds),
        '-Wl,--eh-frame-hdr',
    ],
    link_depends: [shim_map, shim_lds],

    include_directories: includes_libos,
    install: true,
    install_dir: pkglibdir,
)
