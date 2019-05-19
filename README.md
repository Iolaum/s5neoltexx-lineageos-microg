# Custom Build Instructions for s5neoltexx

We will use the [docker image provided by the lineageos4microg team](https://github.com/lineageos4microg/docker-lineage-cicd) and alter accordingly the instructions provided.

## Re-building for Galaxy S5 Neo


As there is no official support for this device, we first have to include the sources in the source tree through an XML in the 
manifests folder. We will use repositories from the [Exynos7580](https://gitlab.com/Exynos7580/) team.

We also set INCLUDE_PROPRIETARY=false, as the proprietary blobs are already
provided by the vendor repositories (so we don't have to include the TheMuppets repo).

We add a repository for the required pre-built packages and another one that includes the bromite system webview which is more privacy friendly
than android's default chromium webview.

We add the needed repositories in our `s5neolte.xml` file and we move the manifest file to `/home/$USER/lineageos/manifests`.

Now we can just run the build like it was officially supported:

```
$ docker run \
     -e "BRANCH_NAME=lineage-15.1" \
     -e "DEVICE_LIST=s5neolte" \
     -e "SIGN_BUILDS=true" \
     -e "SIGNATURE_SPOOFING=restricted" \
     -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension additional_repos OpenWeatherMapWeatherProvider RemoteDroidGuard GsamBattery MatLog" \
     -e "INCLUDE_PROPRIETARY=false" \
     -v "/home/$USER/lineageos/src:/srv/src" \
     -v "/home/$USER/lineageos/zips:/srv/zips" \
     -v "/home/$USER/lineageos/logs:/srv/logs" \
     -v "/home/$USER/lineageos/cache:/srv/ccache" \
     -v "/home/$USER/lineageos/keys:/srv/keys" \
     -v "/home/$USER/lineageos/manifests:/srv/local_manifests" \
     -e "RELEASE_TYPE=CUSTOM_LINEAGE_MICROG" \
     -e "USER_NAME=Nikolaos Perrakis" \
     -e "USER_MAIL=nikperrakis@gmail.com" \
     -e "KEYS_SUBJECT=/C=UK/L=Nottingham/O=LineageOS/emailAddress=nikperrakis@gmail.com" \
     --cpuset-cpus="2" \
     lineageos4microg/docker-lineage-cicd
Set cache size limit to 50.0 GB
>> [Sun May 19 00:27:06 UTC 2019] Branch:  lineage-15.1
>> [Sun May 19 00:27:06 UTC 2019] Devices: s5neolte,
>> [Sun May 19 00:27:07 UTC 2019] (Re)initializing branch repository
>> [Sun May 19 00:27:08 UTC 2019] Copying '/srv/local_manifests/*.xml' to '.repo/local_manifests/'
>> [Sun May 19 00:27:08 UTC 2019] Syncing branch repository
>> [Sun May 19 00:28:43 UTC 2019] Applying the restricted signature spoofing patch (based on android_frameworks_base-O.patch) to frameworks/base
>> [Sun May 19 00:28:43 UTC 2019] Setting "CUSTOM_LINEAGE_MICROG" as release type
>> [Sun May 19 00:28:43 UTC 2019] Adding OTA URL overlay (for custom URL )
>> [Sun May 19 00:28:43 UTC 2019] Adding custom packages (GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension additional_repos OpenWeatherMapWeatherProvider RemoteDroidGuard GsamBattery MatLog)
>> [Sun May 19 00:28:43 UTC 2019] Adding keys path (/srv/keys)
>> [Sun May 19 00:28:43 UTC 2019] Using OpenJDK 8
>> [Sun May 19 00:28:43 UTC 2019] Preparing build environment
>> [Sun May 19 00:28:45 UTC 2019] Starting build for s5neolte, lineage-15.1 branch
>> [Sun May 19 02:10:26 UTC 2019] Moving build artifacts for s5neolte to '/srv/zips/s5neolte'
>> [Sun May 19 02:10:28 UTC 2019] Finishing build for s5neolte
>> [Sun May 19 02:10:28 UTC 2019] Cleaning source dir for device s5neolte
```

The [resulting image](https://www.androidfilehost.com/?fid=1395089523397969224) has been uploaded to androidfilehost.
