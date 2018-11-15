# Custom Build Instructions for s5neoltexx

We will use the [docker image provided by the lineageos4microg team](https://github.com/lineageos4microg/docker-lineage-cicd) and alter accordingly the instructions provided.

## Re-building for Galaxy S5 Neo


As there is no official support for this device, we first have to include the sources in the source tree through an XML in the 
/home/user/manifests folder. From [this](https://github.com/Exynos7580/) GitHub team we get the links of:

    Device tree: https://github.com/Exynos7580/android_device_samsung_s5neoltexx
    Common Tree: https://github.com/Exynos7580/android_device_samsung_exynos7580-common
    Kernel: https://github.com/Exynos7580/android_device_samsung_exynos7580-common
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



We also want to include our microg custom packages so, like before, create an XML (for
example /home/user/manifests/custom_packages.xml) with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="lineageos4microg/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="master" />
</manifest>
```

We move all 3 manifest files to `/home/$USER/lineageos/manifests`.

We also set INCLUDE_PROPRIETARY=false, as the proprietary blobs are already
provided by the vendor repositories (so we don't have to include the TheMuppets repo).


Now we can just run the build like it was officially supported:

```
$ docker run \
    -e "BRANCH_NAME=lineage-15.1" \
    -e "DEVICE_LIST=s5neoltexx" \
    -e "SIGN_BUILDS=true" \
    -e "SIGNATURE_SPOOFING=restricted" \
    -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \
    -e "INCLUDE_PROPRIETARY=false" \
    -v "/home/$USER/lineageos/src:/srv/src" \
    -v "/home/$USER/lineageos/zips:/srv/zips" \
    -v "/home/$USER/lineageos/logs:/srv/logs" \
    -v "/home/$USER/lineageos/cache:/srv/ccache" \
    -v "/home/$USER/lineageos/keys:/srv/keys" \
    -v "/home/$USER/lineageos/manifests:/srv/local_manifests" \
    -e "RELEASE_TYPE=UNOFFICIAL-LINEAGE-MICROG" \
    -e "USER_NAME=SAMPLEUSER" \
    -e "USER_MAIL=SAMPLE@email.com" \
    -e "KEYS_SUBJECT=/C=UN/L=HQ/O=LineageOS/emailAddress=SAMPLE@email.com" \
    lineageos4microg/docker-lineage-cicd
Unable to find image 'lineageos4microg/docker-lineage-cicd:latest' locally
latest: Pulling from lineageos4microg/docker-lineage-cicd
05d1a5232b46: Pull complete 
121eda1015c1: Pull complete 
2c1e53be5dc4: Pull complete 
aa7885f50626: Pull complete 
c2aad5ce7ba5: Pull complete 
2fd8501f1bec: Pull complete 
d835fee635b3: Pull complete 
3b5a8a7bb36e: Pull complete 
a64ae4750f88: Pull complete 
9c56e74bab58: Pull complete 
0b70ea949478: Pull complete 
06ada5d0c2f8: Pull complete 
cf24a32a6bcf: Pull complete 
Digest: sha256:3180b45f3444d05c3e524042cffad01de02f6859126d908bbeed3d52afc09758
Status: Downloaded newer image for lineageos4microg/docker-lineage-cicd:latest
Set cache size limit to 50.0 GB
>> [Tue Nov 13 20:20:13 UTC 2018] SIGN_BUILDS = true but empty $KEYS_DIR, generating new keys
>> [Tue Nov 13 20:20:13 UTC 2018]  Generating releasekey...
>> [Tue Nov 13 20:20:13 UTC 2018]  Generating platform...
>> [Tue Nov 13 20:20:13 UTC 2018]  Generating shared...
>> [Tue Nov 13 20:20:13 UTC 2018]  Generating media...
>> [Tue Nov 13 20:20:13 UTC 2018] Branch:  lineage-15.1
>> [Tue Nov 13 20:20:13 UTC 2018] Devices: s5neoltexx,
>> [Tue Nov 13 20:20:13 UTC 2018] (Re)initializing branch repository
>> [Tue Nov 13 20:20:23 UTC 2018] Copying '/srv/local_manifests/*.xml' to '.repo/local_manifests/'
>> [Tue Nov 13 20:20:23 UTC 2018] Syncing branch repository
>> [Wed Nov 14 01:49:43 UTC 2018] Applying the restricted signature spoofing patch (based on android_frameworks_base-O.patch) to frameworks/base
>> [Wed Nov 14 01:49:43 UTC 2018] Setting "UNOFFICIAL-LINEAGE-MICROG" as release type
>> [Wed Nov 14 01:49:43 UTC 2018] Adding OTA URL overlay (for custom URL )
>> [Wed Nov 14 01:49:43 UTC 2018] Adding custom packages (GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension )
>> [Wed Nov 14 01:49:43 UTC 2018] Adding keys path (/srv/keys)
>> [Wed Nov 14 01:49:43 UTC 2018] Using OpenJDK 8
>> [Wed Nov 14 01:49:43 UTC 2018] Preparing build environment
>> [Wed Nov 14 01:49:44 UTC 2018] Syncing branch repository
>> [Wed Nov 14 01:51:07 UTC 2018] Starting build for s5neoltexx, lineage-15.1 branch
>> [Wed Nov 14 04:01:21 UTC 2018] Moving build artifacts for s5neoltexx to '/srv/zips/s5neoltexx'
>> [Wed Nov 14 04:01:23 UTC 2018] Finishing build for s5neoltexx
>> [Wed Nov 14 04:01:23 UTC 2018] Cleaning source dir for device s5neoltexx

```

The [resulting image](https://www.androidfilehost.com/?fid=11410963190603846763) has been uploaded to androidfilehost.
