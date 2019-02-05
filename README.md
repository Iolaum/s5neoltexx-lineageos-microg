# Custom Build Instructions for s5neoltexx

We will use the [docker image provided by the lineageos4microg team](https://github.com/lineageos4microg/docker-lineage-cicd) and alter accordingly the instructions provided.

## Re-building for Galaxy S5 Neo


As there is no official support for this device, we first have to include the sources in the source tree through an XML in the 
/home/user/manifests folder. From [this](https://github.com/Exynos7580/) GitHub team we get the links of:

    Device tree: https://github.com/Exynos7580/android_device_samsung_s5neoltexx
    Common Tree: https://github.com/Exynos7580/android_device_samsung_exynos7580-common
    Kernel: https://github.com/Exynos7580/android_kernel_samsung_exynos7580-common
    Vendor blobs: https://github.com/Exynos7580/android_vendor_samsung_exynos7580-common

Then, with the help of lineage.dependencies from the 
[device tree](https://github.com/Exynos7580/android_device_samsung_s5neoltexx/blob/lineage-15.1/lineage.dependencies)
and the [common tree](https://github.com/Exynos7580/android_device_samsung_exynos7580-common/blob/lineage-15.1/lineage.dependencies)
we create the needed XML files. However the Exynos7580 team has done that for us at the [local_manifest repository](https://github.com/Exynos7580/local_manifests).

Hence we have the following two xml files.

1. `s5neoltexx.xml`:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="Exynos7580/android_device_samsung_s5neoltexx" path="device/samsung/s5neoltexx" remote="github" revision="lineage-15.1" />
  <project name="Exynos7580/android_vendor_samsung_s5neoltexx" path="vendor/samsung/s5neoltexx" remote="github" revision="lineage-15.1" />
</manifest>
```


2. `exynos7580-common.xml`:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="Exynos7580/android_device_samsung_exynos7580-common" path="device/samsung/exynos7580-common" remote="github" revision="lineage-15.1" />
  <project name="Exynos7580/android_kernel_samsung_exynos7580-common" path="kernel/samsung/exynos7580-common" remote="github" revision="lineage-15.1" />
  <project name="Exynos7580/android_vendor_samsung_exynos7580-common" path="vendor/samsung/exynos7580-common" remote="github" revision="lineage-15.1" />
  <project name="Exynos7580/android_hardware_samsung_slsi-cm_exynos7580" path="hardware/samsung_slsi-cm/exynos7580" remote="github" revision="lineage-15.1" />
  <project name="Exynos7580/android_external_protobuf-compat-2.6" path="external/protobuf-compat-2.6" remote="github" revision="lineage-15.1" />
  <project name="LineageOS/android_packages_resources_devicesettings" path="packages/resources/devicesettings" remote="github" revision="lineage-15.1" />
  <project name="LineageOS/android_packages_apps_FlipFlap" path="packages/apps/FlipFlap" remote="github" revision="lineage-15.1" />
  <project name="TeamNexus/android_hardware_samsung_slsi-cm_exynos" path="hardware/samsung_slsi-cm/exynos" remote="github" revision="nx-8.1" />
  <project name="TeamNexus/android_hardware_samsung_slsi-cm_exynos5" path="hardware/samsung_slsi-cm/exynos5" remote="github" revision="nx-8.1" />
  <project name="TeamNexus/android_hardware_samsung_slsi-cm_openmax" path="hardware/samsung_slsi-cm/openmax" remote="github" revision="nx-8.1" />
  <project name="Exynos7580/android_hardware_samsung" path="hardware/samsung" remote="github" revision="lineage-15.1" />
</manifest>
```


3. `custom_packages.xml`:

We also want to include our microg custom packages so. Because we want to modify the available packages we fork the [upstream repository](https://github.com/lineageos4microg/android_prebuilts_prebuiltapks) and modify it to include the following modifications:

- Changed default GmsCore build to the one [provided by Nanolx through his repository](https://nanolx.org/fdroid/). (I used fdroid from another device to download it and copied it through adb).
- Added RemoteDroidGuard apk as a prebuilt priviliged app. Apk sourced from [flawedworld's PR](https://github.com/lineageos4microg/android_prebuilts_prebuiltapks/pull/9) that's taken from Nanodroid flashable zip.
- Added Gsam Battery app from playstore - through apkmirror download. I did this so that the app has the needed permissions to capture battery usage instead of having to give them afterwards through adb or root.
- Added MatLog Libre app to help with capturing logs. In this case as well the app needs an extra permission through adb, it's easier to include it in the build. Downloaded the app from fdroid's website.

To use those modification we create an XML with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="Iolaum/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="iolaum" />
</manifest>
```

4. `custom_repos.xml`:

In order for our image to run properly we need to apply [this patch](https://review.lineageos.org/#/c/LineageOS/android_hardware_interfaces/+/206140/).
Because this has not been merged upstream we create our own [fork](https://github.com/Iolaum/android_hardware_interfaces) of the repository and apply it
on a [custom branch](https://github.com/Iolaum/android_hardware_interfaces/tree/lineage-15.1-s5neoltexx). Therefore we create the XML file needed to replace the
upstream repository with our custom one:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <remove-project name="LineageOS/android_hardware_interfaces" />
  <project name="Iolaum/android_hardware_interfaces" path="hardware/interfaces" remote="github" revision="lineage-15.1-s5neoltexx" />
</manifest>
```


We move all 4 manifest files to `/home/$USER/lineageos/manifests`.

We also set INCLUDE_PROPRIETARY=false, as the proprietary blobs are already
provided by the vendor repositories (so we don't have to include the TheMuppets repo).


Now we can just run the build like it was officially supported:

```
$ docker run \
>      -e "BRANCH_NAME=lineage-15.1" \
>      -e "DEVICE_LIST=s5neoltexx" \
>      -e "SIGN_BUILDS=true" \
>      -e "SIGNATURE_SPOOFING=restricted" \
>      -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension OpenWeatherMapWeatherProvider RemoteDroidGuard GsamBattery MatLog" \
>      -e "INCLUDE_PROPRIETARY=false" \
>      -v "/home/$USER/lineageos/src:/srv/src" \
>      -v "/home/$USER/lineageos/zips:/srv/zips" \
>      -v "/home/$USER/lineageos/logs:/srv/logs" \
>      -v "/home/$USER/lineageos/cache:/srv/ccache" \
>      -v "/home/$USER/lineageos/keys:/srv/keys" \
>      -v "/home/$USER/lineageos/manifests:/srv/local_manifests" \
>      -e "RELEASE_TYPE=CUSTOM_LINEAGE_MICROG" \
>      -e "USER_NAME=Nikolaos Perrakis" \
>      -e "USER_MAIL=nikperrakis@gmail.com" \
>      -e "KEYS_SUBJECT=/C=UK/L=Nottingham/O=LineageOS/emailAddress=nikperrakis@gmail.com" \
>      lineageos4microg/docker-lineage-cicd
Set cache size limit to 50.0 GB
>> [Mon Feb  4 00:10:05 UTC 2019] Branch:  lineage-15.1
>> [Mon Feb  4 00:10:05 UTC 2019] Devices: s5neoltexx,
>> [Mon Feb  4 00:10:06 UTC 2019] (Re)initializing branch repository
>> [Mon Feb  4 00:10:08 UTC 2019] Copying '/srv/local_manifests/*.xml' to '.repo/local_manifests/'
>> [Mon Feb  4 00:10:08 UTC 2019] Syncing branch repository
>> [Mon Feb  4 00:11:14 UTC 2019] Applying the restricted signature spoofing patch (based on android_frameworks_base-O.patch) to frameworks/base
>> [Mon Feb  4 00:11:14 UTC 2019] Setting "CUSTOM_LINEAGE_MICROG" as release type
>> [Mon Feb  4 00:11:14 UTC 2019] Adding OTA URL overlay (for custom URL )
>> [Mon Feb  4 00:11:14 UTC 2019] Adding custom packages (GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension OpenWeatherMapWeatherProvider RemoteDroidGuard GsamBattery MatLog)
>> [Mon Feb  4 00:11:14 UTC 2019] Adding keys path (/srv/keys)
>> [Mon Feb  4 00:11:14 UTC 2019] Using OpenJDK 8
>> [Mon Feb  4 00:11:15 UTC 2019] Preparing build environment
>> [Mon Feb  4 00:11:15 UTC 2019] Starting build for s5neoltexx, lineage-15.1 branch
>> [Mon Feb  4 00:59:53 UTC 2019] Moving build artifacts for s5neoltexx to '/srv/zips/s5neoltexx'
>> [Mon Feb  4 00:59:55 UTC 2019] Finishing build for s5neoltexx
>> [Mon Feb  4 00:59:55 UTC 2019] Cleaning source dir for device s5neoltexx
```

The [resulting image](https://www.androidfilehost.com/?fid=11410963190603915175) has been uploaded to androidfilehost.
