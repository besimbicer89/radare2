# handle capstone dependency
capstone_dep = dependency('capstone', version: '>=3.0.4', required: false)
if not capstone_dep.found() or not get_option('use_sys_capstone')
  message('Use bundled capstone')

  if get_option('capstone_in_builddir')
    capstone_path = join_paths(meson.current_build_dir(), 'capstone')
  else
    capstone_path = 'capstone'
  endif

  res = run_command(py3_exe, '-c', '__import__("sys").exit(__import__("os").path.exists("@0@"))'.format(capstone_path))
  if res.returncode() == 0
    if not git_exe.found()
      error('Cannot load capstone library. Either provide capstone in ./shlr/capstone or install git, so it can be downloaded')
    endif

    # NOTE: when you update CS_TIP or CS_BRA, also update them in shlr/Makefile
    use_capstone5 = get_option('use_capstone5')
    if use_capstone5
      CS_TIP = 'd956ceddabcbe544ee8f074ba75c86c6381b34e6'
      CS_BRA = 'next'
    else
      CS_TIP = '2edae851d9fee511a57d4da32d5acecedd95d7ed'
      CS_BRA = 'v4'
    endif

    capstone_git_user = 'aquynh'

    message('Cloning capstone ' + CS_BRA + ' branch, commit ' + CS_TIP + ', into ' + capstone_path)
    git_cmd = 'clone -b @0@ https://github.com/@1@/capstone.git @2@'.format(CS_BRA, capstone_git_user, capstone_path)
    clone_cmd = run_command(git_exe, git_cmd.split())
    if clone_cmd.returncode() != 0
      error('Cannot execute git clone command')
    endif

    reset_cmd_str = '-C @0@ reset --hard @1@'.format(capstone_path, CS_TIP)
    reset_cmd = run_command(git_exe, reset_cmd_str.split())
    if reset_cmd.returncode() != 0
      error('Cannot execute git reset command')
    endif

    patches_files = [
      'capstone-calloc.patch',
      'fix-x86-16.patch',
      'sparc-crash.patch',
      'sstream-null.patch'
    ]

    message('Patching capstone with radare\'s patches')
    foreach file : patches_files
      patch_path = join_paths(meson.current_source_dir(), 'capstone-patches', file)
      patch_cmd_str = '-C @0@ apply -p1 @1@'.format(capstone_path, patch_path)
      patch_cmd = run_command(git_exe, patch_cmd_str.split())
      if patch_cmd.returncode() != 0
        warning('Cannot apply patch ' + file)
      endif
    endforeach
  endif

  rel_cs_files = [
    'arch/AArch64/AArch64BaseInfo.c',
    'arch/AArch64/AArch64Disassembler.c',
    'arch/AArch64/AArch64InstPrinter.c',
    'arch/AArch64/AArch64Mapping.c',
    'arch/AArch64/AArch64Module.c',
    'arch/ARM/ARMDisassembler.c',
    'arch/ARM/ARMInstPrinter.c',
    'arch/ARM/ARMMapping.c',
    'arch/ARM/ARMModule.c',
    'arch/M680X/M680XDisassembler.c',
    'arch/M680X/M680XInstPrinter.c',
    'arch/M680X/M680XModule.c',
    'arch/M68K/M68KDisassembler.c',
    'arch/M68K/M68KInstPrinter.c',
    'arch/M68K/M68KModule.c',
    'arch/Mips/MipsDisassembler.c',
    'arch/Mips/MipsInstPrinter.c',
    'arch/Mips/MipsMapping.c',
    'arch/Mips/MipsModule.c',
    'arch/PowerPC/PPCDisassembler.c',
    'arch/PowerPC/PPCInstPrinter.c',
    'arch/PowerPC/PPCMapping.c',
    'arch/PowerPC/PPCModule.c',
    'arch/Sparc/SparcDisassembler.c',
    'arch/Sparc/SparcInstPrinter.c',
    'arch/Sparc/SparcMapping.c',
    'arch/Sparc/SparcModule.c',
    'arch/SystemZ/SystemZDisassembler.c',
    'arch/SystemZ/SystemZInstPrinter.c',
    'arch/SystemZ/SystemZMapping.c',
    'arch/SystemZ/SystemZMCTargetDesc.c',
    'arch/SystemZ/SystemZModule.c',
    'arch/TMS320C64x/TMS320C64xDisassembler.c',
    'arch/TMS320C64x/TMS320C64xInstPrinter.c',
    'arch/TMS320C64x/TMS320C64xMapping.c',
    'arch/TMS320C64x/TMS320C64xModule.c',
    'arch/X86/X86ATTInstPrinter.c',
    'arch/X86/X86Disassembler.c',
    'arch/X86/X86DisassemblerDecoder.c',
    'arch/X86/X86IntelInstPrinter.c',
    'arch/X86/X86Mapping.c',
    'arch/X86/X86Module.c',
    'arch/XCore/XCoreDisassembler.c',
    'arch/XCore/XCoreInstPrinter.c',
    'arch/XCore/XCoreMapping.c',
    'arch/XCore/XCoreModule.c',
    'cs.c',
    'MCInst.c',
    'MCInstrDesc.c',
    'MCRegisterInfo.c',
    'SStream.c',
    'utils.c',
  ]

  cs_files = []
  foreach rel_cs_file : rel_cs_files
    cs_files += [join_paths(capstone_path, rel_cs_file)]
  endforeach
  if get_option('use_capstone5')
    cs_files += [join_paths(capstone_path, 'arch/X86/X86InstPrinterCommon.c')]
  endif

  capstone_includes = [platform_inc, include_directories('capstone/include')]

  libr2capstone = static_library('r2capstone', cs_files,
    c_args: [
      '-DCAPSTONE_X86_ATT_DISABLE_NO',
      '-DCAPSTONE_X86_REDUCE_NO',
      '-DCAPSTONE_USE_SYS_DYN_MEM',
      '-DCAPSTONE_DIET_NO',
      '-DCAPSTONE_HAS_ARM',
      '-DCAPSTONE_HAS_ARM64',
      '-DCAPSTONE_HAS_M68K',
      '-DCAPSTONE_HAS_M680X',
      '-DCAPSTONE_HAS_MIPS',
      '-DCAPSTONE_HAS_POWERPC',
      '-DCAPSTONE_HAS_SPARC',
      '-DCAPSTONE_HAS_SYSZ',
      '-DCAPSTONE_HAS_X86',
      '-DCAPSTONE_HAS_XCORE',
      '-DCAPSTONE_HAS_TMS320C64X',
    ],
    include_directories: capstone_includes,
    implicit_include_directories: false
  )

  capstone_dep = declare_dependency(
    link_with: libr2capstone,
    include_directories: capstone_includes
  )
else
  message('Use system-provided capstone library')
endif


# handle sdb dependency
# NOTE: copying most of the stuff from sdb to here for the moment, since we
# should use subpackages to handle this well
sdb_files = [
  'sdb/src/array.c',
  'sdb/src/base64.c',
  'sdb/src/buffer.c',
  'sdb/src/set.c',
  'sdb/src/cdb.c',
  'sdb/src/cdb_make.c',
  'sdb/src/dict.c',
  'sdb/src/diff.c',
  'sdb/src/disk.c',
  'sdb/src/fmt.c',
  'sdb/src/ht_uu.c',
  'sdb/src/ht_up.c',
  'sdb/src/ht_pp.c',
  'sdb/src/journal.c',
  'sdb/src/json.c',
  'sdb/src/lock.c',
  'sdb/src/ls.c',
  # 'sdb/src/main.c',
  'sdb/src/match.c',
  'sdb/src/ns.c',
  'sdb/src/num.c',
  'sdb/src/query.c',
  'sdb/src/sdb.c',
  'sdb/src/sdbht.c',
  'sdb/src/util.c'
]

sdb_inc = [platform_inc, include_directories('sdb/src')]

libr2sdb = static_library('r2sdb', sdb_files,
  include_directories: sdb_inc,
  implicit_include_directories: false,
  c_args: host_machine.system() == 'windows' ? '-DSDB_API=__declspec(dllexport)' : [],
)

sdb_dep = declare_dependency(
  link_whole: libr2sdb,
  include_directories: sdb_inc
)

# Create sdb_version.h
r = run_command(py3_exe, 'sys/sdb_version.py', 'sdb/config.mk')
if r.returncode() == 0
  sdb_version = r.stdout().strip().split('\n')[0]
else
  sdb_version = 'unknown'
endif
run_command(py3_exe, '-c', 'with open("sdb/src/sdb_version.h", "w") as f: f.write("#define SDB_VERSION \"' + sdb_version + '\"")')

sdb_exe = executable('sdb', ['sdb/src/main.c'] + sdb_files,
  include_directories: [
    include_directories(['sdb/src'])
  ],
  implicit_include_directories: false,
  native: true,
)

sdb_gen_cmd_script = '''#script
import os
import sys
sdb_exe = r'@0@'
outfile = sys.argv[1]
infiles = sys.argv[2:]
if os.name == 'nt':
    # DO NOT replace chr(92) below with a backslash
    infiles, sdb_exe, outfile = map(lambda x: x.replace('/', chr(92)), [' '.join(infiles), sdb_exe, outfile])
    os.system('type {} | {} {} ='.format(infiles, sdb_exe, outfile))
else:
    infiles = ' '.join(infiles)
    os.system('cat {} | {} {} ='.format(infiles, sdb_exe, outfile))
'''.format(sdb_exe.full_path())

sdb_gen_cmd = [
  py3_exe,
  '-c',
  sdb_gen_cmd_script,
  '@OUTPUT@',
  '@INPUT@'
]

# handle tree-sitter
tree_sitter_dep = dependency('tree-sitter', required: false)
if not tree_sitter_dep.found() or not get_option('use_sys_tree_sitter')
  message('Use bundled tree-sitter')
  tree_sitter_path = 'tree-sitter'
  tree_sitter_vc_path = 'tree-sitter.vc'
  if get_option('tree-sitter-sync')
    if not git_exe.found()
      error('Cannot sync tree-sitter library. Either provide tree-sitter in ./shlr/tree-sitter or install git, so it can be downloaded')
    endif

    # NOTE: when you update TS_TIP or TS_BRA, also update them in shlr/Makefile
    TS_TIP = '9a82dcc666d06617cbab3061467075019fae0b0d'
    TS_BRA = 'master'

    message('Deleting existing directories @0@ and @1@'.format(tree_sitter_vc_path, tree_sitter_path))
    res = run_command('rm', '-rf @0@ @1@'.format(tree_sitter_vc_path, tree_sitter_path).split())
    message('Cloning tree-sitter ' + TS_BRA + ' branch, commit ' + TS_TIP + ', into ' + tree_sitter_vc_path)
    git_cmd = 'clone -b @0@ https://github.com/tree-sitter/tree-sitter.git @1@'.format(TS_BRA, tree_sitter_vc_path)
    clone_cmd = run_command(git_exe, git_cmd.split())
    if clone_cmd.returncode() != 0
      error('Cannot execute git clone command')
    endif

    reset_cmd_str = '-C @0@ reset --hard @1@'.format(tree_sitter_vc_path, TS_TIP)
    reset_cmd = run_command(git_exe, reset_cmd_str.split())
    if reset_cmd.returncode() != 0
      error('Cannot execute git reset command')
    endif

    message('Copying files from @0@ to @1@'.format(tree_sitter_vc_path, tree_sitter_path))
    res = run_command('mkdir',  '-p @0@/lib'.format(tree_sitter_path).split())
    res = run_command('cp',  '-r @0@/lib/src @1@/lib'.format(tree_sitter_vc_path, tree_sitter_path).split())
    res = run_command('cp',  '-r @0@/lib/include @1@/lib'.format(tree_sitter_vc_path, tree_sitter_path).split())
    message('Deleting @0@'.format(tree_sitter_vc_path))
    res = run_command('rm', '-rf @0@'.format(tree_sitter_vc_path).split())
  endif

  tree_sitter_files = [
    join_paths(tree_sitter_path, 'lib/src/lib.c'),
  ]

  tree_sitter_inc = [platform_inc, include_directories('tree-sitter/lib/src'), include_directories('tree-sitter/lib/include')]

  libtree_sitter = static_library('tree_sitter', tree_sitter_files,
    include_directories: tree_sitter_inc,
    implicit_include_directories: false,
    c_args: ['-std=c99']
  )

  tree_sitter_dep = declare_dependency(
    link_with: libtree_sitter,
    include_directories: tree_sitter_inc
  )
else
  message('Use system-provided tree-sitter library')
endif


# new radare2 shell parser
shell_parser_path = 'radare2-shell-parser'
shell_parser_files = [
  join_paths(shell_parser_path, 'src/parser.c'),
  join_paths(shell_parser_path, 'src/scanner.c'),
]

shell_parser_inc = [platform_inc, include_directories('radare2-shell-parser/src/tree_sitter')]

libshell_parser = static_library('shell_parser', shell_parser_files,
  include_directories: shell_parser_inc,
  dependencies: tree_sitter_dep.partial_dependency(includes: true),
  implicit_include_directories: true
)

shell_parser_dep = declare_dependency(
  link_with: libshell_parser,
  include_directories: shell_parser_inc,
  dependencies: tree_sitter_dep
)


# handle bochs dependency
bochs_files = [
  'bochs/src/libbochs.c'
]

bochs_inc = [platform_inc, include_directories('bochs/include')]

libr2bochs = static_library('r2bochs', bochs_files,
  include_directories: bochs_inc,
  implicit_include_directories: false
)

bochs_dep = declare_dependency(
  link_with: libr2bochs,
  include_directories: bochs_inc
)


# handle java dependency
java_files = [
  'java/class.c',
  'java/code.c',
  'java/dsojson.c',
  'java/ops.c',
  #'java/main.c',
]

java_inc = [platform_inc, include_directories('java')]

libr2java = static_library('r2java', java_files,
  include_directories: java_inc,
  implicit_include_directories: false
)

java_dep = declare_dependency(
  link_with: libr2java,
  include_directories: java_inc
)


# handle qnx dependency
qnx_files = [
  'qnx/src/core.c',
  'qnx/src/libqnxr.c',
  'qnx/src/packet.c',
  'qnx/src/sigutil.c',
  'qnx/src/utils.c',
]

qnx_inc = [platform_inc, include_directories('qnx/include')]

libr2qnx = static_library('r2qnx', qnx_files,
  include_directories: qnx_inc,
  implicit_include_directories: false
)

qnx_dep = declare_dependency(
  link_with: libr2qnx,
  include_directories: qnx_inc
)


# handle tcc dependency (heavily modified version)
tcc_files = [
  'tcc/libtcc.c',
  'tcc/tccgen.c',
  'tcc/tccpp.c'
]

tcc_inc = [platform_inc, include_directories('tcc')]

libr2tcc = static_library('r2tcc', tcc_files,
  include_directories: tcc_inc,
  implicit_include_directories: false
)

tcc_dep = declare_dependency(
  link_with: libr2tcc,
  include_directories: tcc_inc
)


# handle spp dependency
spp_files = [
  'spp/spp.c'
]

spp_inc = [platform_inc, include_directories('spp')]

libr2spp = static_library('r2spp', spp_files,
  include_directories: spp_inc,
  c_args: ['-DHAVE_R_UTIL'],
  implicit_include_directories: false
)

spp_dep = declare_dependency(
  link_with: libr2spp,
  include_directories: spp_inc
)


# handle lz4 dependency
lz4_dep = dependency('liblz4', required: false)
if not lz4_dep.found() or not get_option('use_sys_lz4')
  message('Use bundled lz4')
  lz4_files = [
    'lz4/lz4.c',
  ]

  lz4_inc = [platform_inc, include_directories('lz4')]

  libr2lz4 = static_library('r2lz4', lz4_files,
    include_directories: lz4_inc,
    implicit_include_directories: false
  )

  lz4_dep = declare_dependency(
    link_with: libr2lz4,
    include_directories: lz4_inc
  )
else
  message('Use system-provided lz4 library')
endif


# handle zlib dependency
zlib_dep = dependency('zlib', required: false)
if not zlib_dep.found() or not get_option('use_sys_zlib')
  message('Use bundled zlib')

  zlib_files = [
    'zip/zlib/adler32.c',
    'zip/zlib/compress.c',
    'zip/zlib/crc32.c',
    'zip/zlib/deflate.c',
    'zip/zlib/gzclose.c',
    'zip/zlib/gzlib.c',
    'zip/zlib/gzread.c',
    'zip/zlib/gzwrite.c',
    'zip/zlib/infback.c',
    'zip/zlib/inffast.c',
    'zip/zlib/inflate.c',
    'zip/zlib/inftrees.c',
    'zip/zlib/trees.c',
    'zip/zlib/uncompr.c',
    'zip/zlib/zutil.c'
  ]

  zlib_inc = [platform_inc, include_directories('zip/zlib')]

  libr2zlib = static_library('r2zlib', zlib_files,
    include_directories: zlib_inc,
    implicit_include_directories: false
  )

  zlib_dep = declare_dependency(
    link_with: libr2zlib,
    include_directories: zlib_inc
  )
else
  message('Use system-provided zlib library')
endif


# handle zip dependency
zip_dep = dependency('libzip', required: false)
if not zip_dep.found() or not get_option('use_sys_zip')
  message('Use bundled zip')

  zip_files = [
    'zip/zip/zip_add.c',
    'zip/zip/zip_add_dir.c',
    'zip/zip/zip_add_entry.c',
    'zip/zip/zip_close.c',
    'zip/zip/zip_delete.c',
    'zip/zip/zip_dir_add.c',
    'zip/zip/zip_dirent.c',
    'zip/zip/zip_discard.c',
    'zip/zip/zip_entry.c',
    'zip/zip/zip_err_str.c',
    'zip/zip/zip_error.c',
    #'zip/zip/zip_error_clear.c',
    'zip/zip/zip_error_get.c',
    'zip/zip/zip_error_get_sys_type.c',
    'zip/zip/zip_error_strerror.c',
    'zip/zip/zip_error_to_str.c',
    'zip/zip/zip_extra_field.c',
    'zip/zip/zip_extra_field_api.c',
    'zip/zip/zip_fclose.c',
    'zip/zip/zip_fdopen.c',
    'zip/zip/zip_file_add.c',
    #'zip/zip/zip_file_error_clear.c',
    'zip/zip/zip_file_error_get.c',
    'zip/zip/zip_file_get_comment.c',
    'zip/zip/zip_file_get_offset.c',
    'zip/zip/zip_file_rename.c',
    'zip/zip/zip_file_replace.c',
    'zip/zip/zip_file_set_comment.c',
    'zip/zip/zip_file_strerror.c',
    'zip/zip/zip_filerange_crc.c',
    'zip/zip/zip_fopen.c',
    'zip/zip/zip_fopen_encrypted.c',
    'zip/zip/zip_fopen_index.c',
    'zip/zip/zip_fopen_index_encrypted.c',
    'zip/zip/zip_fread.c',
    'zip/zip/zip_get_archive_comment.c',
    'zip/zip/zip_get_archive_flag.c',
    'zip/zip/zip_get_compression_implementation.c',
    'zip/zip/zip_get_encryption_implementation.c',
    'zip/zip/zip_get_file_comment.c',
    'zip/zip/zip_get_name.c',
    'zip/zip/zip_get_num_entries.c',
    'zip/zip/zip_get_num_files.c',
    'zip/zip/zip_name_locate.c',
    'zip/zip/zip_new.c',
    'zip/zip/zip_open.c',
    'zip/zip/zip_rename.c',
    'zip/zip/zip_replace.c',
    'zip/zip/zip_set_archive_comment.c',
    'zip/zip/zip_set_archive_flag.c',
    'zip/zip/zip_set_default_password.c',
    'zip/zip/zip_set_file_comment.c',
    'zip/zip/zip_set_file_compression.c',
    'zip/zip/zip_set_name.c',
    'zip/zip/zip_source_buffer.c',
    'zip/zip/zip_source_close.c',
    'zip/zip/zip_source_crc.c',
    'zip/zip/zip_source_deflate.c',
    'zip/zip/zip_source_error.c',
    'zip/zip/zip_source_file.c',
    'zip/zip/zip_source_filep.c',
    'zip/zip/zip_source_free.c',
    'zip/zip/zip_source_function.c',
    'zip/zip/zip_source_layered.c',
    'zip/zip/zip_source_open.c',
    'zip/zip/zip_source_pkware.c',
    'zip/zip/zip_source_pop.c',
    'zip/zip/zip_source_read.c',
    'zip/zip/zip_source_stat.c',
    'zip/zip/zip_source_window.c',
    'zip/zip/zip_source_zip.c',
    'zip/zip/zip_source_zip_new.c',
    'zip/zip/zip_stat.c',
    'zip/zip/zip_stat_index.c',
    'zip/zip/zip_stat_init.c',
    'zip/zip/zip_strerror.c',
    'zip/zip/zip_string.c',
    'zip/zip/zip_unchange.c',
    'zip/zip/zip_unchange_all.c',
    'zip/zip/zip_unchange_archive.c',
    'zip/zip/zip_unchange_data.c',
    'zip/zip/zip_utf-8.c'
  ]

  zip_inc = [platform_inc, include_directories('zip/include')]

  libr2zip = static_library('r2zip', zip_files,
    include_directories: zip_inc,
    implicit_include_directories: false,
    dependencies: zlib_dep
  )

  zip_dep = declare_dependency(
    link_with: libr2zip,
    include_directories: zip_inc
  )
else
  message('Use system-provided zip library')
endif


# handle grub dependency
grub_files = [
  #'grub/fs/affs.c',
  #'grub/fs/afs.c',
  #'grub/fs/afs_be.c',
  #'grub/fs/befs.c',
  #'grub/fs/befs_be.c',
  #'grub/fs/btrfs.c',
  'grub/fs/cpio.c',
  'grub/fs/ext2.c',
  'grub/fs/fat.c',
  'grub/fs/fb.c',
  'grub/fs/fshelp.c',
  'grub/fs/hfs.c',
  'grub/fs/hfsplus.c',
  'grub/fs/iso9660.c',
  'grub/fs/jfs.c',
  'grub/fs/minix.c',
  #'grub/fs/minix2.c',
  #'grub/fs/nilfs2.c',
  'grub/fs/ntfs.c',
  'grub/fs/ntfscomp.c',
  'grub/fs/reiserfs.c',
  'grub/fs/sfs.c',
  'grub/fs/tar.c',
  'grub/fs/udf.c',
  'grub/fs/ufs.c',
  'grub/fs/ufs2.c',
  'grub/fs/xfs.c',
  'grub/grubfs.c',
  'grub/kern/device.c',
  'grub/kern/disk.c',
  'grub/kern/env.c',
  'grub/kern/err.c',
  'grub/kern/file.c',
  'grub/kern/fs.c',
  'grub/kern/list.c',
  'grub/kern/misc.c',
  'grub/kern/mm.c',
  'grub/kern/partition.c',
  'grub/kern/term.c',
  'grub/kern/time.c',
  #'grub/main.c',
  #'grub/partmap/acorn.c',
  'grub/partmap/amiga.c',
  'grub/partmap/apple.c',
  'grub/partmap/bsdlabel.c',
  'grub/partmap/gpt.c',
  'grub/partmap/msdos.c',
  'grub/partmap/sun.c',
  'grub/partmap/sunpc.c',
]

grub_inc = [platform_inc, include_directories('grub', 'grub/include')]

libr2grub = static_library('r2grub', grub_files,
  include_directories: grub_inc,
  implicit_include_directories: false
)

grub_dep = declare_dependency(
  link_with: libr2grub,
  include_directories: grub_inc
)


# handle windbg dependency
windbg_files = [
  'windbg/iob_pipe.c',
  'windbg/kd.c',
  'windbg/transport.c',
  'windbg/windbg.c',
]

windbg_inc = [platform_inc, include_directories('windbg')]

libr2windbg = static_library('r2windbg', windbg_files,
  include_directories: windbg_inc,
  implicit_include_directories: false
)

windbg_dep = declare_dependency(
  link_with: libr2windbg,
  include_directories: windbg_inc
)


# handle ar dependency
ar_files = [
  'ar/ar.c'
]

ar_inc = [platform_inc, include_directories(['ar'])]

libr2ar = static_library('r2ar', ar_files,
  include_directories: ar_inc,
  implicit_include_directories: false
)

ar_dep = declare_dependency(
  link_with: libr2ar,
  include_directories: ar_inc
)


# handle ptrace-wrap dependency
if use_ptrace_wrap
  subdir('ptrace-wrap')
endif

if host_machine.system() == 'windows'
  subdir('w32dbg_wrap')
endif

# handle mpc dependency
mpc_files = [
  'mpc/mpc.c'
]

mpc_inc = [platform_inc, include_directories(['mpc'])]

libmpc = static_library('r2mpc', mpc_files,
  include_directories: mpc_inc,
  implicit_include_directories: false
)

mpc_dep = declare_dependency(
  link_with: libmpc,
  include_directories: mpc_inc
)

# handle yxml dependency
yxml_files = [
  'yxml/yxml.c'
]

yxml_inc = [platform_inc, include_directories(['yxml'])]

libyxml = static_library('r2yxml', yxml_files,
  include_directories: yxml_inc,
  implicit_include_directories: false
)

yxml_dep = declare_dependency(
  link_with: libyxml,
  include_directories: yxml_inc
)
