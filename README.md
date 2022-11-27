![alt text][logo]

[logo]:https://crdroid.net/img/logo.png "crDroid Android"

## 1. Grabbing the source ##

[Repo](http://source.android.com/source/developing.html) is a tool provided by Google that
simplifies using [Git](http://git-scm.com/book) in the context of the Android source.

### 1.1 Installing dependencies and Repo ###

Several packages are needed in order to build crDroid
```
sudo apt install bc bison build-essential ccache curl flex g++-multilib gcc-multilib git gnupg gperf imagemagick lib32ncurses5-dev lib32readline-dev lib32z1-dev liblz4-tool libncurses5 libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-gtk3-dev libxml2 libxml2-utils lzop pngcrush rsync schedtool squashfs-tools xsltproc zip zlib1g-dev
```

Install Repo tool
```bash
# Make a directory where Repo will be stored and add it to the path
$ mkdir ~/bin
$ PATH=~/bin:$PATH

# Download Repo itself
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo

# Make Repo executable
$ chmod a+x ~/bin/repo
```

### 1.2 Initializing Repo ###

```bash
# Create a directory for the source files
# This can be located anywhere (as long as the fs is case-sensitive)
$ mkdir crDroid
$ cd crDroid

# Install Repo in the created directory
$ repo init -u https://github.com/awakened1712/android.git -b 13.0
$ git clone https://github.com/awakened1712/local_manifests.git .repo/local_manifests -b android-13
```

This is what you will run each time you want to pull in upstream changes. Keep in mind that on your
first run, it is expected to take a while as it will download all the required Android source files
and their change histories.

```bash
# Let Repo take care of all the hard work
$ repo sync -c --force-sync --no-clone-bundle --no-tags
```

```bash
# Run to prepare our devices list
$ . build/envsetup.sh
# ... now run
$ brunch lineage_beyond1lte-userdebug
```

## 2. Fixup ##
Open `/vendor/gapps/arm64/arm64-vendor.mk` and delete the following lines
```
PRODUCT_COPY_FILES += \
    vendor/gapps/arm64/proprietary/product/lib/libjni_latinimegoogle.so:$(TARGET_COPY_OUT_PRODUCT)/lib/libjni_latinimegoogle.so \
    vendor/gapps/arm64/proprietary/product/lib64/libjni_latinimegoogle.so:$(TARGET_COPY_OUT_PRODUCT)/lib64/libjni_latinimegoogle.so
```

## 3. Upload ##
Create OAuth client ID for TV and Limited input devices to generate client_id and client_secret
Save the below file as curlgoogle into `~/bin`
```python
#!/usr/bin/python
'''
A quick python script to automate curl->googledrive interfacing
This should require nothing more than the system python version and curl. Written for python2.7 (with 3 in mind).
Dan Ellis 2020
'''
import os,sys,json
##############################
#Owner information goes here!#
##############################
name = sys.argv[1]
client_id= '243729134676-0dp6u46f8ssg505sh0n32s8g1n9egg9l.apps.googleusercontent.com'
client_secret='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'
##############################

cmd1 = json.loads(os.popen('curl -d "client_id=%s&scope=https://www.googleapis.com/auth/drive.file" https://oauth2.googleapis.com/device/code'%client_id).read())
str(input('\n Enter %(user_code)s\n\n at %(verification_url)s \n\n Then hit Enter to continue.'%cmd1))
str(input('(twice)'))
cmd2 = json.loads(os.popen(('curl -d client_id=%s -d client_secret=%s -d device_code=%s -d grant_type=urn~~3Aietf~~3Aparams~~3Aoauth~~3Agrant-type~~3Adevice_code https://accounts.google.com/o/oauth2/token'%(client_id,client_secret,cmd1['device_code'])).replace('~~','%')).read())
print(cmd2)
cmd4 = os.popen('''
curl -X POST -L \
    -H "Authorization: Bearer %s" \
    -F "metadata={name :\'%s\'};type=application/json;charset=UTF-8" \
    -F "file=@%s;type=application/zip" \
    "https://www.googleapis.com/upload/drive/v3/files?uploadType=multipart"
    '''%(cmd2["access_token"],name,name)).read()
print(cmd4)
print('end')
```
Run the upload script
```bash
cd FILE_PATH
curlgoogle FILE
```
