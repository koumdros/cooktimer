import os
import sys
from os.path import join;

wxFolder = wxCfg = None

# this function was copied from http://aspn.activestate.com/ASPN/Cookbook/Python/Recipe/52296
def GetCommandOutput2(command):
    child = os.popen(command)
    data = child.read()
    err = child.close()
    if err:
		raise RuntimeError, '%s failed w/ exit code %d' % (command, err)
    return data

def RunWxConfig(parameters):
	assert(wxFolder != None and wxCfg != None);
	return GetCommandOutput2("wx-config \"--prefix=%s\" \"--wxcfg=%s\" %s" % (wxFolder, wxCfg, parameters));

opts = Options()
opts.Add(BoolOption('release', 'Set to 1 for release build', 0))

if ("WXWIN" in os.environ):
	wxFolder = os.environ["WXWIN"];
else:
	wxFolder = "C:\\wxWidgets-2.6.4";

env = None
if (ARGUMENTS.get("MSVS_VERSION", None) == None):
	env = Environment(tools = ['msvc', 'mslink'], options = opts, ENV = os.environ)	
else:
	env = Environment(options = opts, ENV = os.environ, MSVS_VERSION=ARGUMENTS.get("MSVS_VERSION", None))
Help(opts.GenerateHelpText(env))

debug = (env['release'] == 0)

psdk_libs = ["user32", "kernel32", "shell32", "shlwapi", "comctl32", "gdi32", "ole32", "wsock32", "advapi32", "userenv", "Comdlg32", "Rpcrt4", "Netapi32", "uuid", "Version", "oleaut32"]
				  
env.Append(CCFLAGS = ["-EHsc"])	# enable exceptions

wxCfg = ("vc_lib/mswud", "vc_lib/mswu")[debug == False];

cxxFlags = RunWxConfig("--cxxflags")
libs = RunWxConfig("--libs")

env.AppendUnique(CCFLAGS = cxxFlags.split())
env.AppendUnique(LINKFLAGS = libs.split())

env.Append(LIBS = [psdk_libs]);

#env.Append(RCFLAGS = RunWxConfig("--rcflags").split());
env.Append(RCFLAGS = "-I" + join(wxFolder, "include"));

resources = env.RES(["resources.rc"]);

buildResult = env.Program("build/CookTimer", ["CookTimerFrame.cpp", "main.cpp",
						resources])

if (int(float(env["MSVS_VERSION"][0:2])) >= 8):
	# Add a post-build step to embed the manifest using mt.exe
	# The number at the end of the line indicates the file type (1: EXE; 2:DLL).
	env.AddPostAction(buildResult, 'mt.exe -nologo -manifest ${TARGET}.manifest -outputresource:$TARGET;1')

if (debug):
	env.Append(CPPDEFINES = ["__WXDEBUG__"]);
	env.Append(CCFLAGS = ["-Zi"])
	env.Append(CCFLAGS = ["-RTCcsu", "-GS"]) # generate stack corruption checks
	env.Append(CCFLAGS = ["-MD"]);
	env.Append(LINKFLAGS = ["-debug"]);
else:
	# todo: enable before releasing.
	#env.Append(CPPDEFINES = ["NDEBUG"])
	env.Append(CCFLAGS = ["-Ox", "-G6", "-GA"])	# optimization options
	env.Append(CCFLAGS = ["-MD"]);
	env.Append(LINKFLAGS = ["-release", "-OPT:REF", "-OPT:NOWIN98", "-OPT:ICF=32"]);
