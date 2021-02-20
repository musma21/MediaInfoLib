# MediaInfoLib README

MediaInfo(Lib) is a convenient unified display of the most relevant technical and tag data for video and audio files.

[![Build Status](https://travis-ci.org/MediaArea/MediaInfoLib.svg?branch=master)](https://travis-ci.org/MediaArea/MediaInfoLib)
[![Build status](https://ci.appveyor.com/api/projects/status/enael8oersp6nntd/branch/master?svg=true)](https://ci.appveyor.com/project/MediaArea/mediainfolib/branch/master)

MediaInfoLib - https://github.com/MediaArea/MediaInfoLib  
Copyright (c) MediaArea.net SARL. All Rights Reserved.

This program is freeware under BSD-2-Clause license conditions.  
See License.html for more information

# Some changes for access to Azure Blob storage


MediaInfoLib library sets "x-ms-range" HTTP header to send HTTP/1.1 range request ([RFC 7233](https://tools.ietf.org/html/rfc7233)). Azure Blob storage accepts the range request only when "x-ms-version" is set to 2011-08-18 or higher ([Specifying the range header for Blob service operations](https://docs.microsoft.com/en-us/rest/api/storageservices/specifying-the-range-header-for-blob-service-operations)). However, with current Mediainfo, there's no way to specify the custom HTTP header. 

Adding the below single line would be a quick fix (Reader_libcurl::Format_Test_PerParser() in Reader_libcurl.cpp) : 

```C++
Curl_Data->HttpHeader = curl_slist_append (Curl_Data->HttpHeader, "x-ms-version: 2020-04-08");
```


That being said, still it would be better to add an command line option to set the custom HTTP header, so here, let me give it a try to add the new feature. Once practical progress be made, I will file an issue to the main repo ([MediaArea/MediaInfoLib](https://github.com/MediaArea/MediaInfoLib/)). Meanwhile, just for tracing purpose, ```MEDIAINFO_DEBUG``` preprocessor directive has been replaced with ```__MEDIAINFO_DEBUG_AZUREBLOB__``` in some files - ```Reader_File.cpp, Reader_libcurl.cpp, and MediaInfoList.cpp```; however, the directive should be effective only within this fetch of mine.


## How to build under macOS and Linux 

```sh
cd $BUILD_DIR
git clone https://github.com/musma21/MediaInfoLib.git
cd MediaInfoLib/Project/GNU/Library
./autogen.sh
./configure --disable-static --with-libcurl --enable-debug-azureblob
make
```

## Note from Andrew SungHee Yi (musma21@msn.com)

* Feedback to Azure documentation ([Specifying the range header for Blob service operations](https://docs.microsoft.com/en-us/rest/api/storageservices/specifying-the-range-header-for-blob-service-operations))  
```Please explicitly docuement "x-ms-version" is mandatory to send the range request. And better helpful if you note the link to the latest version of Azure Storage services. (https://docs.microsoft.com/en-us/rest/api/storageservices/versioning-for-the-azure-storage-services)``` 

* How to reproduce the issue mentioned above 
Let's say your container is open to the public : 
```sh
mediainfo http://your_stroage_account.blob.core.windows.net/your_container/mediafile.mp4
```
And compare the output from the below curl commands : 
```sh
curl -o /dev/null -s -v http://your_storage_account.blob.core.windows.net/your_container/mediafile.mp4 -r 12699-
```
```sh
curl -o /dev/null -s -v http://your_storage_account.blob.core.windows.net/your_container/mediafile.mp4 -r 12699- `
-H "x-ms-version: 2020-04-08"
```
