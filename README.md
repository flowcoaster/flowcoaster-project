To install flowcoaster, please follow the following instructions:

* download android 4.3_r1:

```bash
mkdir -p ~/tdroid/tdroid-4.3_r1
cd ~/tdroid/tdroid-4.3_r1
repo init -u https://android.googlesource.com/platform/manifest -b android-4.3_r1
repo sync
```

* ensure android build works:

```bash
source build/envsetup.sh
lunch 1
make -j4
```

* If everything went through, prepare to download flowcoaster sources. For this purpose, edit ```.repo/local_manifests/local_manifest.xml``` and add following content

```xml
<manifest>
  <remote name="github" fetch="git://github.com"/>
  <remove-project name="platform/build"/>
  <project path="build" remote="github" name="flowcoaster/android_platform_build" revision="master">
    <copyfile src="core/root.mk" dest="Makefile" />
  </project>
  <remove-project name="platform/dalvik"/>
  <project path="dalvik" remote="github" name="flowcoaster/android_platform_dalvik" revision="master"/>
  <remove-project name="platform/libcore"/>
  <project path="libcore" remote="github" name="TaintDroid/android_platform_libcore" revision="taintdroid-4.3_r1"/>
  <remove-project name="platform/frameworks/base"/>
  <project path="frameworks/base" remote="github" name="flowcoaster/android_platform_frameworks_base" revision="master"/>
  <remove-project name="platform/frameworks/native"/>
  <project path="frameworks/native" remote="github" name="flowcoaster/android_platform_frameworks_native" revision="master"/>
  <remove-project name="platform/frameworks/opt/telephony"/>
  <project path="frameworks/opt/telephony" remote="github" name="TaintDroid/android_platform_frameworks_opt_telephony" revision="taintdroid-4.3_r1"/>
  <remove-project name="platform/system/vold"/>
  <project path="system/vold" remote="github" name="TaintDroid/android_platform_system_vold" revision="taintdroid-4.3_r1"/>
  <remove-project name="platform/system/core"/>
  <project path="system/core" remote="github" name="flowcoaster/android_platform_system_core" revision="master"/>
  <remove-project name="platform/system/extras"/>
  <project path="system/extras" remote="github" name="flowcoaster/android_platform_system_extras" revision="master"/>
  <remove-project name="platform/libnativehelper"/>
  <project path="libnativehelper" remote="github" name="flowcoaster/android_platform_libnativehelper" revision="master"/>
  <remove-project name="platform/external/valgrind"/>
  <project path="external/valgrind" remote="github" name="flowcoaster/android_platform_external_valgrind" revision="master"/>
  <remove-project name="device/samsung/manta"/>
  <project path="device/samsung/manta" remote="github" name="TaintDroid/device_samsung_manta" revision="taintdroid-4.3_r1"/>
  <remove-project name="device/samsung/tuna"/>
  <project path="device/samsung/tuna" remote="github" name="TaintDroid/android_device_samsung_tuna" revision="taintdroid-4.3_r1"/>
  <project path="packages/apps/TaintDroidNotify" remote="github" name="TaintDroid/android_platform_packages_apps_TaintDroidNotify" revision="taintdroid-4.3_r1"/>
  <project path="packages/apps/jniapitester" remote="github" name="flowcoaster/android_platform_packages_apps_jniapitester" revision="master" />
</manifest>
```

* to get the sources and to make them editable, execute the following commands

```
repo sync
repo forall libcore frameworks/opt/telephony system/vold device/samsung/manta device/samsung/tuna \
       packages/apps/TaintDroidNotify -c 'git checkout -b taintdroid-4.3_r1 --track github/taintdroid-4.3_r1 && git pull'

repo forall build dalvik frameworks/base frameworks/native system/core system/extras libnativehelper build packages/apps/jniapitester external/valgrind -c 'git checkout -b master --track github/master && git pull'
```

* edit/create buildspec.mk to set compile flags

```
# Enable core taint tracking logic (always add this)
WITH_TAINT_TRACKING := true

# Enable taint tracking for ODEX files (always add this)
WITH_TAINT_ODEX := true

# Enable taint tracking in the "fast" (aka ASM) interpreter (recommended)
WITH_TAINT_FAST := true

# Enable additional output for tracking JNI usage (not recommended)
#TAINT_JNI_LOG := true

# Enable byte-granularity tracking for IPC parcels
WITH_TAINT_BYTE_PARCEL := true
```

* remake sources to compile flowcoaster

```bash
make clean
. build/envsetup.sh
lunch 1
make -j4
```