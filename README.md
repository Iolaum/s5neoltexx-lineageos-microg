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


3. `custom_packages.xml`:

We also want to include our microg custom packages so, like before, create an XML with this content:

```
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="lineageos4microg/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="master" />
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
    -e "RELEASE_TYPE=UNOFFICIAL_LINEAGE_MICROG" \
    -e "USER_NAME=SAMPLEUSER" \
    -e "USER_MAIL=SAMPLE@email.com" \
    -e "KEYS_SUBJECT=/C=UN/L=HQ/O=LineageOS/emailAddress=SAMPLE@email.com" \
    lineageos4microg/docker-lineage-cicd
# ...

```

The [resulting image](https://www.androidfilehost.com/?fid=11410963190603853070) has been uploaded to androidfilehost.
