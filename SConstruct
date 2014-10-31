import os, sys
from subprocess import *

def getGitDesc():
  return Popen('git describe --abbrev=8 --tags --always', stdout=PIPE, shell=True).stdout.read ().strip ()


GIT_DESC = getGitDesc ()
print "building " + GIT_DESC + ".."
env = Environment ()

# Verbose / Non-verbose output{{{
colors = {}
colors['cyan']   = '\033[96m'
colors['purple'] = '\033[95m'
colors['blue']   = '\033[94m'
colors['green']  = '\033[92m'
colors['yellow'] = '\033[93m'
colors['red']    = '\033[91m'
colors['end']    = '\033[0m'

#If the output is not a terminal, remove the colors
if not sys.stdout.isatty():
   for key, value in colors.iteritems():
      colors[key] = ''

compile_source_message = '%scompiling %s==> %s$SOURCE%s' % \
   (colors['blue'], colors['purple'], colors['yellow'], colors['end'])

compile_shared_source_message = '%scompiling shared %s==> %s$SOURCE%s' % \
   (colors['blue'], colors['purple'], colors['yellow'], colors['end'])

link_program_message = '%slinking Program %s==> %s$TARGET%s' % \
   (colors['red'], colors['purple'], colors['yellow'], colors['end'])

link_library_message = '%slinking Static Library %s==> %s$TARGET%s' % \
   (colors['red'], colors['purple'], colors['yellow'], colors['end'])

ranlib_library_message = '%sranlib Library %s==> %s$TARGET%s' % \
   (colors['red'], colors['purple'], colors['yellow'], colors['end'])

link_shared_library_message = '%slinking Shared Library %s==> %s$TARGET%s' % \
   (colors['red'], colors['purple'], colors['yellow'], colors['end'])

java_compile_source_message = '%scompiling %s==> %s$SOURCE%s' % \
   (colors['blue'], colors['purple'], colors['yellow'], colors['end'])

java_library_message = '%screating Java Archive %s==> %s$TARGET%s' % \
   (colors['red'], colors['purple'], colors['yellow'], colors['end'])

AddOption("--verbose",action="store_true", dest="verbose_flag",default=False,help="verbose output")
if not GetOption("verbose_flag"):
  env["CXXCOMSTR"] = compile_source_message,
  env["CCCOMSTR"] = compile_source_message,
  env["SHCCCOMSTR"] = compile_shared_source_message,
  env["SHCXXCOMSTR"] = compile_shared_source_message,
  env["ARCOMSTR"] = link_library_message,
  env["RANLIBCOMSTR"] = ranlib_library_message,
  env["SHLINKCOMSTR"] = link_shared_library_message,
  env["LINKCOMSTR"] = link_program_message,
  env["JARCOMSTR"] = java_library_message,
  env["JAVACCOMSTR"] = java_compile_source_message,

# Verbose }}}

# set up compiler and linker from env variables
if os.environ.has_key('CC'):
  env['CC'] = os.environ['CC']
if os.environ.has_key('CFLAGS'):
  env['CCFLAGS'] += SCons.Util.CLVar(os.environ['CFLAGS'])
if os.environ.has_key('CXX'):
  env['CXX'] = os.environ['CXX']
if os.environ.has_key('CXXFLAGS'):
  env['CXXFLAGS'] += SCons.Util.CLVar(os.environ['CXXFLAGS'])
if os.environ.has_key('CPPFLAGS'):
  env['CCFLAGS'] += SCons.Util.CLVar(os.environ['CPPFLAGS'])
if os.environ.has_key('LDFLAGS'):
  env['LINKFLAGS'] += SCons.Util.CLVar(os.environ['LDFLAGS'])


def CheckPKGConfig(context, version):
  context.Message( 'Checking for pkg-config... ' )
  ret = context.TryAction('pkg-config --atleast-pkgconfig-version=%s' % version)[0]
  context.Result( ret )
  return ret

def CheckPKG(context, name):
  context.Message( 'Checking for %s... ' % name )
  ret = context.TryAction('pkg-config --exists \'%s\'' % name)[0]
  context.Result( ret )
  return ret


conf = Configure(env, custom_tests = { 'CheckPKGConfig' : CheckPKGConfig,
                                       'CheckPKG' : CheckPKG })

if not conf.CheckPKGConfig('0.15.0'):
  print 'pkg-config >= 0.15.0 not found.'
  Exit(1)

if not conf.CheckPKG('glibmm-2.4'):
  print "glibmm-2.4 not found."
  Exit (1)

if not conf.CheckPKG('gmime-2.6'):
  print "gmime-2.6 not found."
  Exit (1)

if not conf.CheckLibWithHeader ('notmuch', 'notmuch.h', 'c'):
  print "notmuch does not seem to be installed."
  Exit (1)

# external libraries
env.ParseConfig ('pkg-config --libs --cflags glibmm-2.4')
env.ParseConfig ('pkg-config --libs --cflags gmime-2.6')

if not conf.CheckLib ('boost_filesystem', language = 'c++'):
  print "boost_filesystem does not seem to be installed."
  Exit (1)

if not conf.CheckLib ('boost_system', language = 'c++'):
  print "boost_system does not seem to be installed."
  Exit (1)

if not conf.CheckLib ('boost_program_options', language = 'c++'):
  print "boost_program_options does not seem to be installed."
  Exit (1)

if not conf.CheckLib ('boost_date_time', language = 'c++'):
  print "boost_date_time does not seem to be installed."
  Exit (1)

libs   = ['notmuch',
          'boost_filesystem',
          'boost_system',
          'boost_program_options',
          'boost_date_time']

env.AppendUnique (LIBS = libs)
env.AppendUnique (CPPFLAGS = ['-g', '-Wall', '-std=c++11', '-pthread'] )

# write version file
print ("writing version.hh..")
vfd = open ('version.hh', 'w')
vfd.write ("# pragma once\n")
vfd.write ("# define GIT_DESC \"%s\"\n\n" % GIT_DESC)
vfd.close ()

env.Append (CPPPATH = 'src')

env = conf.Finish ()

env.Program (source = 'tfsync.cc', target = 'tfsync')
env.Program (source = 'keywsync.cc', target = 'keywsync')
env.Alias ('build', ['tfsync', 'keywsync'])

nmenv = Environment (LIBS = ['notmuch'])
nmenv.Program ('notmuch_get_revision', source = [ 'notmuch_get_revision.cc' ])

