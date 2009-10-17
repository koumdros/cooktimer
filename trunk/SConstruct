import os
import sys
from os.path import join;

sys.path.append('.')
from wxconfig import *

wxFolder = None

opts = Options()
opts.Add(BoolOption('release', 'Set to 1 for release build', 0))

if ("WXWIN" in os.environ):
	wxFolder = os.environ["WXWIN"];
else:
	wxFolder = "C:\\wxWidgets-2.6.4";

env = None
if (ARGUMENTS.get("MSVS_VERSION", None) == None):
	if sys.platform == 'win32':
		env = Environment(tools = ['msvc', 'mslink'], options = opts, ENV = os.environ)	
	else:
		env = Environment(options = opts, ENV = os.environ)
else:
	env = Environment(options = opts, ENV = os.environ, MSVS_VERSION=ARGUMENTS.get("MSVS_VERSION", None))

Help(opts.GenerateHelpText(env))

### setup variables needed, mainly to support cross-compilation
env['build_platform']=env['PLATFORM']
env['target_platform']=env['PLATFORM']

env['wxconfig']='wx-config'

compiler = env['CXX']

# HACK for scons 1.2 on Windows where env['CXX'] == '$CC'
while compiler[0] == '$':
	compiler = env[compiler[1:]];
# end HACK

# some helper variables
targetPlatformIsWindows = (env['target_platform'] == 'win32');
compilerIsVC = (compiler == 'cl');
compilerIsGccLike = not compilerIsVC;

if not env.GetOption('clean'):
	if compilerIsVC:
		print "Compiling using Visual C++...";
	elif compilerIsGccLike:
		print "Compiling using '%s', assuming GCC-like compiler" % env['CXX'];

debug = (env['release'] == 0)

### detect wxWidgets
conf = Configure(env, custom_tests = {'CheckWXConfig' : CheckWXConfig })

# CheckWXConfig(version, componentlist, debug)
#          version: string with minimum version ('x.y')
#    componentlist: list of components needed. This was introduced with
#                   wxWidgets 2.6 if I'm correct. You'll usually need
#                   ['adv', 'core', 'base'], in this order. If you use
#                   wxGLCanvas, prepend 'gl' like below.
if not conf.CheckWXConfig('2.6', ['core', 'base', 'adv'], debug):
	print 'Error finding wxWidgets library.'

env = conf.Finish()

### target
ParseWXConfig(env)

resources = [];

psdk_libs = ["user32", "kernel32", "shell32", "shlwapi", "comctl32", "gdi32", "ole32", "wsock32", "advapi32", "userenv", "Comdlg32", "Rpcrt4", "Netapi32", "uuid", "Version", "oleaut32"]

if targetPlatformIsWindows:
	env.Append(LIBS = [psdk_libs]);

	resources = env.RES(["resources.rc"]);

if compilerIsVC:
	env.Append(CCFLAGS = ["-EHsc"])	# enable exceptions

	if (debug):
		env.Append(CXXFLAGS = ["-Zi"])
		env.Append(CXXFLAGS = ["-RTCcsu", "-GS"]) # generate stack corruption checks
		env.Append(CXXFLAGS = ["-MTd"]);
		env.Append(LINKFLAGS = ["-debug"]);
	else:
		env.Append(CPPDEFINES = ["NDEBUG"])
		env.Append(CXXFLAGS = ["-Ox", "-GA"])	# optimization options
		env.Append(CXXFLAGS = ["-MT"]);
		env.Append(LINKFLAGS = ["-release", "-OPT:REF", "-OPT:ICF=32"]);
elif compilerIsGccLike:
	if (debug):
		env.Append(CXXFLAGS = ["-g"]);
	else:
		env.Append(CXXFLAGS = ["-O2"]);
	
buildResult = env.Program("build/CookTimer", ["CookTimerFrame.cpp", "main.cpp",
						resources])

# On Non-Windows platforms, place the wav file in the output directory because
# it's not embedded into the exe
if not targetPlatformIsWindows:
	env.Command('build/alarm.wav', 'alarm.wav', Copy('$TARGET', '$SOURCE'))

# post actions
if compilerIsVC and int(float(env["MSVS_VERSION"][0:2])) >= 8:
	# Add a post-build step to embed the manifest using mt.exe
	# The number at the end of the line indicates the file type (1: EXE; 2:DLL).
	env.AddPostAction(buildResult, 'mt.exe -nologo -manifest ${TARGET}.manifest -outputresource:$TARGET;1')
