PhotoStation Upload (Lightroom plugin)
======================================
Version 3.0<br>
2015/06/16<br>
Copyright(c) 2015, Martin Messmer<br>

Overview
=========
PhotoStation Upload is a Lightroom Publish and Export Service Provider Plugin. It adds a new Publish Service called "PhotoStation Upload" and a new Export target also called "PhotoStation Upload" to the "Export" dialog. Both the Publish service as well as the Export service enable the export of pictures and videos from Lightroom directly to a Synology PhotoStation. It will not only upload the selected photos/videos but also create and upload all required thumbnails and accompanying additional video files.

This plugin uses the same converters and the same upload API as the official "Synology PhotoStation Uploader" tool, but will not use the Uploader itself. The upload API is http-based, so you have to specify the target PhotoStation by protocol (http/https) and servename (IP@, hostname, FQDN). For some of the publish functions the plugin also needs access to the FileStation Web-API, which is also http(s)-based but uses a different port (the admin port) and probably a different account.

Requirements
=============

* Windows OS or Mac, tested with:
	- Windows 7,  Windows 8.1
	- MacOS 10.7.5
	- MacOS 10.10
* Lightroom, 
  tested with:
	- Lr 5.7 and 5.7.1 (Mac and Win)
  reportedly works with:
	- Lr 4.4, 6.0, 6.0.1, 6.1
* Synology PhotoStation, tested with:
	PhotoStation 6
* For Publish mode: Synology FileStation WebAPI (reachable via admin port)
* Synology PhotoStation Uploader, required components:
	- ImageMagick/convert.exe
	- ffmpeg/ffmpeg.exe
	- ffmpeg/qt-faststart.exe

Installation
=============
- unzip the downloaded archive
- copy the subdirectory "PhotoStation_upload.lrplugin" to the machine where Lightroom is installed
- In Lightroom:
	*File* --\> *Plugin Manager* --\> *Add*: Enter the path to the directory 
		"PhotoStation_upload.lrplugin" 

Description
============

Export vs. Publish Service - general remarks
---------------------------------------------
Exporting in Lightroom is a simple one-time processe: you define the photos to export by selecting the photos or folders to export in library view and then choose "Export". Lightroom does not keep track of exports, thus if you want to re-export changed or added photos or remove deleted photos form the target (e.g. a PhotoStation album) later, you will have to keep track yourself for those changes, addtions or deletions.

Publishing in Lightroom on the other hand is meant for synchonizing local photo collections with a remote target (e.g. a PhotoStation album). To publish a photo collection you have to do two things:

- define the settings for the Publish Service
- define the Published Collection and the settings for that Published Collection

As soon as you've done this, Lightroom will keep track of which photo from the collection has to be published, re-published (when it was modified locally) or deleted. Besides that basic functions, some publish services can also re-import certain infos such as tags, comments or ratings back from the publish target.

Export vs. Publish Service - PhotoStation Upload
-------------------------------------------------
The main functionality of PhotoStation Upload is basicly the same in Export and in Publish mode: uploading pictures/videos to a Synology PhotoStation. On top of this the Publish mode also implements the basic publishing function, so that Lr can keep track of added, modified and deleted photos/videos. 

Due to the different handling of exporting and publishing in Lightroom the Export and the Publish dialog of PhotoStation Upload have some but not all of their settings in common. 

The Export dialog includes settings for:

a) the installation path of the Synology<br>
b) the target PhotoStation (server, login, Standard/Personal PhotoStation)<br>
c) -- no --<br>
d) target Album within the target PhotoStation and Upload method<br>
e) quality parameters for thumbs and additional videos<br>

The Publish dialog on the other hand includes settings for:

a) the installation path of the Synology<br>
b) the target PhotoStation (server, login, Standard/Personal PhotoStation)<br>
c) the FileStation API parameters: (protocol, port, login)<br>
d) -- no --<br>
e) quality parameters for thumbs and additional videos<br>

The Album settings ( d) ) are not stored within the Publish settings but within the Published Collections settings. Therefore, you don't need to define a different Publish Service for each Published Collection you want to publish. In most cases you will only have one Publish Service definition and a bunch of Published Collections below it. An additional Publish Service definition is only required, if you want to upload to a different PhotoStation or if you want to use different upload quality settings.

Export Funtionality
--------------------
- upload to the Standard PhotoStation or to a Personal PhotoStation (make sure the Personal PhotoStation feature is enabled for the given Personal Station owner)

- two different upload methods:
	- flat upload: 
	  upload all selected pictures/videos to a named Album (use the folder name, not the Album name) on the PhotoStation
	  upload all selected pictures/videos to a named Album (use the folder name, not the Album name) on the PhotoStation
	  The named Album may exist on the PhotoStation or may be created during export
	  The root Album is defined by an empty string. In general, Albums are specified by "\<folder\>{/\<folder\>}" (no leading or trailing slashes required)
	- tree mirror upload: 
	  preserves the directory path of each photo/video relative to a given local base path on the PhotoStation below a named target Album.
	  All directories within the source path of the picture/video will be created recursively.
	  The directory tree is mirrored relative to a given local base path. Example:<br>
	  Local base path:	C:\users\john\pictures<br>
	  To Album:			Test<br>
	  Photo to export:	C:\users\john\pictures\2010\10\img1.jpg<br>
	  --\> upload to:	Test/2010/10/img1.jpg<br>
	  In other words:	\<local-base-path\>\\\<relative-path\>\\file -- upload to --\> \<Target Album\>/\<relative-path\>/file<br>

- optimize the upload for PhotoStation 6 by not uploading the THUMB_L thumbnail.

- upload of videos and accompanying videos with a lower resolution.

- hard-rotation od uploaded videos that are soft-rotated or meta-rotated (see Version 2.8 changes).

Publish Funtionality:
--------------------

### All Export Functions are Supported in Publish mode

### Definition of a secondary server (Publish dialog)

You may want to publish to your PhotoStation from at home or via the Internet. 
Therefore, the Publish Service dialog allows you to define two sets of server address settings. 
Switching between the two address settings can be done in the Publish dialog
  
### Support for Published Collections and Published Smart Collections 

No support for Published Collection Sets.
 
### Different Publish options (Published Collection dialog)

- Normal:
  publish unpublished photos to target Album in target PhotoStation 
- CheckExisting:
  Unpublished photos will not be published, but will be checked whether they already exist in the target Album and if so, set them to 'Published'. 
  This operation mode is useful when initializing a new Published Collection: if you have exported the latest version of thoses photos before to the 
  defined target but not through the newly defined Published Collection (e.g. via Export).
  CheckExisting is 15 to 30 times faster than a Normal Publish, since no thumbnail creation and upload is required.
  Note, that CheckExisting can not determine, whether the photo in the target Album is the latest version.
- CheckMoved:
  Check if any photo within a Published Collection has moved locally and if so, mark them to 're-publish'
  If your Published Collection is to be tree-mirrored to the target Album, it is important to notice when a photo was moved locally between directories, since these movements have to be propagated to the target Album (i.e., the photo has to be deleted at the target Album at its old location and re-published at the new location).
  Unfortunately, Lightroom will not mark moved photos for 're-publish'. Therefore, this mode is a workaround for this missing Lr feature.
  To use it, you have to set at least one photo to 're-publish', otherwise you won't be able to push the "Publish" button.
  CheckMoved is very fast (\>100 photos/sec) since it only checks locally whether the local path of a photo has changed in comparison to its published location. There is no communication to the PhotoStation involved.
- deletion of published photos, when deleted locally (from Collection or Library)
- deletion of complete Published Collections
- no support for re-import of comments or ratings
- no support for custom-defined sort order

Additional Funtionality
------------------------
- checks for updates in background when Exporting, Publishing or opening the Plugin section in the Plugin Manager no more than once per day.
  If a new version is available, you'll get an info message after the Export/Publish and also a note in the Plugin Manager section.
  The update check will send the following information to the update server:
	- PhotoStation Upload plugin version
	- Operating system version
	- Lightroom version
	- Lightroom language setting
	- a random unique identifier chosen by the update service
  This helps me keep track of the different environments/combinations the plugin is running in.

Important note
---------------
Passwords entered in the export settings are not stored encrypted, so they might be accessible by other plugins or other people that have access to your system. So, if you mind storing your password in the export settings, you may leave the password field in the export settings empty so that you will be prompted to enter username/password when the export starts.

History
=========

Version 2.2 (initial public release)
-------------------------------------
- Generic upload features:
	- support for http and https
	- support for non-standard ports (specified by a ":portnumber" suffix in the servername setting)
	- support for pathnames incl. blanks and non-standard characters (e.g.umlauts) (via url-encoding)
	- uses the following PhotoStation http-based APIs:
		- Login
		- Create Folder
		- Upload File
	- supports the PhotoStation upload batching mechanism
	- optimization for PhotoStation 6 (no need for THUM_L)

- Folder management
	- support for flat copy and tree copy (incl. directory creation)

- Upload of Photos to PhotoStation:
	- upload of Lr-rendered photos
	- generation (via ImageMagick convert) and upload of all required thumbs

- Upload of Videos to PhotoStation:
	- upload of original or Lr-rendered videos 
	- generation (via ffpmeg and ImageMagick convert) and upload of all required thumbs
	- generation (via ffpmeg) and upload of a PhotoStation-playable low-res video
	- support for "DateTimeOriginal" for videos on PhotoStation 

Version 2.3
------------
- Fixed various (!!) installation / initialization bugs
- Fixed strange field validation behaviour in Export Dialog
- Fixed mis-aligned input fields in Export Dialog
- Added Loglevel configuration to Export Dialog section
- Added: "Goto Logfile" on failures
- Modified thumbnail creation to the "Syno PS Uploader" way:
	slightly slower but higher thumbnail quality (less sharp) (Hint from Uwe)
- Added option "Create Album, if needed"
- Added completion bezel

Version 2.4
------------
- Export Dialog re-design with lots of tooltips
- Support for small (Synology old-style) and large thumbnails (Synology new-style)

Version 2.5
------------
- Configurable thumbnail generation quality (in percent)
- Target album not required in preset; prompt for it before upload starts, if missing 

Version 2.6
------------
- video upload completely reworked
- support for DateTimeOriginal (capture date) in uploaded video
- support for videos with differen aspect ratios (16:9, 4:3, 3:2, ...) 
	- recognizes the video aspect ratio by mpeg dimension tag and by mpeg dar (display aspect ratio) tag
	- generate thumbnails and videos in correct aspect ratio
- support for uploading of original videos in various formats:
	- if file is '\*.mp4', no conversion required, otherwise the original video has to be converted to mp4
- support for uploading of one additional mp4-video in a different (lower) resolution:
	- additional video resolution is configurable separately for different original video resolutions
- fixed video conversion bug under MacOS (2.6.4)
- fixed mis-alignment of other export sections (2.6.5)
- note: make sure to select "Include Video" and Format "Original" in the Video settings section 
	to avoid double transcoding and to preserve	the DateTimeOriginal (capture date) in the uploaded video

Version 2.7
------------
- Bugfix for failed upload when filename includes '( 'or ')', important only for MacOS
- Quicker (15%) upload for PS6 by not generating the Thumb_L which is not uploaded anyway

Version 2.8
------------
Added video rotation support: 
- soft-rotated videos (w/ rotation tag in mpeg header) now get the right (rotated) thumbs
- hard-rotation option for soft-rotated videos for better player compatibility:
  Soft-rotated videos are not rotated in most players, PhotoStation supports soft-rotated 
  videos only by generating an additional hard-rotated flash-video. 
  This may be OK for small videos, but overloads the DiskStation CPU for a period of time. 
  Thus, it is more desirable to hard-rotate the videos on the PC before uploading.
  Hard-rotated videos with (then) potrait orientation work well in VLC, but not at all in 
  MS Media Player. So, if you intend to use MS Media Player, you should stay with the 
  soft-rotated video to see at least a mis-rotated video. 
  In all other cases hard-rotation is probably more feasable for you.
- support for "meta-rotation":
  If you have older mis-rotated videos (like I have lots of from my children's 
  video experiments), these videos typically don't have a rotation indication. 
  So, the described hard-rotation support won't work for those videos. To circumvent this, 
  the Uploader supports rotation indications by metadata maintained in Lr. 
  To tag a desired rotation for a video, simply add one of the following 
  keywords to the video in Lr:
  - Rotate-90	--\> for videos that need 90 degree clockwise rotation
  - Rotate-180	--\> for videos that need 180 degree rotation
  - Rotate-270	--\> for videos that need 90 degree counterclockwise rotation
- support for soft-rotation and hard-rotation for "meta-rotated" (see above) videos 
		
Version 3.0
------------
Added Publish mode

Open issues
============
- issue in PhotoStation: if video aspect ratio is different from video dimension 
  (i.e. sample aspect ratio [sar] different from display aspect ratio [dar]) 
  the galery thumb of the video will be shown with a wrong aspect ratio (= sar)
- publishing requires access to FileStation WebAPI, which is cumbersome, since it requires access to the admin port.
  This may be change sometime in the future, if the PhotoStation WebAPI will be published by Synology
- "Show in PhotoStation" always opens the root of the target PhotoStation, not the root of target Album nor the selecte photo in the target Album
  
Copyright
==========
Copyright(c) 2015, Martin Messmer

PhotoStation Upload is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

PhotoStation Upload is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with PhotoStation Upload.  If not, see <http://www.gnu.org/licenses/>.

PhotoStation Upload uses the following free software to do its job:

- convert.exe,			see: http://www.imagemagick.org/
- ffmpeg.exe, 			see: https://www.ffmpeg.org/
- qt-faststart.exe, 	see: http://multimedia.cx/eggs/improving-qt-faststart/