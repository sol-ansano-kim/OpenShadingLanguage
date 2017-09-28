import excons
import sys
import os
from excons.tools import llvm


env = excons.MakeBaseEnv()

out_basedir = excons.OutputBaseDirectory()
out_incdir = out_basedir + "/include"
out_libdir = out_basedir + "/lib"
llvm_static = excons.GetArgument("llvm-static", 1, int)



OSL_OPTS = {}
OSL_DEPENDENCIES = []
OSL_OPTS["LINKSTATIC"] = "ON"
OSL_OPTS["BUILDSTATIC"] = "OFF"


### dependencies

## boost
boost_rv = excons.cmake.ExternalLibRequire(OSL_OPTS, "boost")
# filesystem wave
if not boost_rv["require"]:
    excons.WarnOnce("boost is require to build OpenShadingLanguage, please provide root directory using 'with-boost=' flag", tool="OSL")
    sys.exit(1)

OSL_OPTS["BOOST_ROOT"] = os.path.dirname(boost_rv["incdir"])


llvm
rv = excons.cmake.ExternalLibRequire(OSL_OPTS, "llvm")

if not rv["require"]:
    excons.PrintOnce("OSL: Build llvm from sources ...")
    excons.Call("llvm")
    OSL_OPTS["LLVM_DIRECTORY"] = out_basedir
    OSL_OPTS["LLVM_STATIC"] = "ON"
    OSL_DEPENDENCIES
else:
    OSL_OPTS["LLVM_DIRECTORY"] = os.path.dirname(rv["incdir"])
    OSL_OPTS["LLVM_STATIC"] = "ON" if llvm_static else "OFF"


oiio_rv = excons.cmake.ExternalLibRequire(OSL_OPTS, "oiio")
if not oiio_rv["require"]:
    oiio_opts = {"oiio-static": 1}
    excons.PrintOnce("OSL: Build oiio from sources ...")
    excons.Call("oiio", overrides=oiio_opts, imp=["OiioPath"])
    OSL_DEPENDENCIES.append(OiioPath(static=True))

    OSL_OPTS["OPENIMAGEIOHOME"] = os.path.dirname(os.path.dirname(OiioPath(static=True)))
else:
    OSL_OPTS["OPENIMAGEIOHOME"] = os.path.dirname(oiio_rv["incdir"])

    openexr_rv = excons.cmake.ExternalLibRequire(OSL_OPTS, "openexr")
    if not openexr_rv["require"]:
        excons.PrintOnce("OSL: Build openexr from sources ...")
        excons.Call("openexr", imp=["IlmImfUtilPath", "IlmImfPath", "IlmThreadPath", "ImathPath", "HalfPath", "IexPath", "IexMathPath"])

        OSL_DEPENDENCIES += [IlmImfUtilPath(static=True), IlmImfPath(static=True), IlmThreadPath(static=True), ImathPath(static=True), HalfPath(static=True), IexPath(static=True), IexMathPath(static=True)]

        openexr_dir = os.path.dirname(os.path.dirname(IlmImfUtilPath(static=True)))
        OSL_OPTS["OPENEXR_CUSTOM_INCLUDE_DIR"] = os.path.join(openexr_dir, "include")
        OSL_OPTS["OPENEXR_CUSTOM_LIB_DIR"] = os.path.join(openexr_dir, "lib")

    else:
        OSL_OPTS["OPENEXR_CUSTOM_INCLUDE_DIR"] = openexr_rv["incdir"]
        OSL_OPTS["OPENEXR_CUSTOM_LIB_DIR"] = openexr_rv["libdir"]

print OSL_DEPENDENCIES


prjs = []

prjs.append({"name": "osl",
             "type": "cmake",
             "cmake-opts": OSL_OPTS,
             "cmake-cfgs": excons.CollectFiles(["src"], patterns=["CMakeLists.txt"], recursive=True) + ["CMakeLists.txt"] + OSL_DEPENDENCIES,
             "cmake-srcs": excons.CollectFiles(["src"], patterns=["*.cpp"], recursive=True),
             "cmake-outputs": map(lambda x: out_incdir + "/OSL/" + os.path.basename(x), excons.glob("src/include/OSL/*.h"))})    

excons.DeclareTargets(env, prjs)
