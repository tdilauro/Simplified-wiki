# What we mirror

* Project Gutenberg .epub files and images
* Cover images from Open Library

# S3

I've created an S3 bucket called 'simplified.nypl.org'. We use S3FS-FUSE to mount the S3 bucket as a filesystem. This lets us use rsync and other filesystem tools on the bucket. Unfortunately, these tools don't work very well because S3 is not a filesystem.

## Installing S3FS

* sudo apt-get install git make g++ autogen automake libfuse-dev libcurl4-openssl-dev libxml2-dev
* git clone https://github.com/s3fs-fuse/s3fs-fuse.git
* cd s3fs-fuse
* ./autogen
* ./configure
* make
* sudo make install

## Mounting the bucket

I mount my bucket in ~/data/mirror. My credentials are kept in ~/data/s3_credentials.txt in the following format:

[Access Key Id]:[Secret Access Key]

* cd ~/data/
* chmod 600 s3_credentials.txt

I can mount the bucket once with this command:

s3fs simplified.nypl.org /home/leonardr/data/mirror -o passwd_file=/home/leonardr/data/s3_credentials.txt

On the production machine I want the bucket to be automounted on startup. So I add this to /etc/fstab:

s3fs#simplified.nypl.org /data/mirror fuse auto,passwd_file=/data/s3_credentials.txt,allow_other,user 0 0

# Gutenberg mirror

We maintain the mirror with rsync as per [these instructions](http://www.gutenberg.org/wiki/Gutenberg:Mirroring_How-To). We are currently interested only in the generated epub documents and the cover images. 

The mirror is stored in mirror/gutenberg.org. This command will update the mirror:

$ rsync --del -av --include="*.epub" --include="*cover*" --filter="-! */" ftp@ftp.ibiblio.org::gutenberg-epub gutenberg.org

The initial mirror will probably hang several times because S3FS does not deal well with uploading thousands of files at once. Once we complete a mirror, updates should be no problem. 