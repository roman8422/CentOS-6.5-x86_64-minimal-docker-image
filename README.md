# CentOS 6.5 x86_64 minimal docker image

## Steps to create:
To create a CentOS 6.5 image we will use [officially recommended](https://docs.docker.com/engine/userguide/eng-image/baseimages/) tool mkimage-yum.sh from Moby project.
We'll use latest official centos:6 docker image. That image comes with repositories for all previous releases disabled by default

- Start our dev container
    ```
    docker container run -it --name dev --rm -v $(pwd):/mnt centos:6
    ```
- Install packages, configure yum, clone moby project:
    ```
    yum -y install git yum-utils                    # install packages
    yum-config-manager --disable \*                 # disable all repos
    yum-config-manager --enable C6.5*               # enable centos6.5 repos
    
    cd /root
    git clone https://github.com/moby/moby/         # clone project moby
    cd /root/moby/contrib

    ```

- Since we don't want to run this image on our dev container, we need to update last 3 lines in mkimage-yum.sh as shown below:
    ```diff
    diff --git a/contrib/mkimage-yum.sh b/contrib/mkimage-yum.sh
    index 9012804..61a0779 100755
    --- a/contrib/mkimage-yum.sh
    +++ b/contrib/mkimage-yum.sh
    @@ -129,8 +129,5 @@ if [ -z "$version" ]; then
         version=$name
     fi
     
    -tar --numeric-owner -c -C "$target" . | docker import - $name:$version
    +tar --numeric-owner -cf /mnt/"$name".tar -C "$target" .
     
    -docker run -i -t --rm $name:$version /bin/bash -c 'echo success'
    -
    -rm -rf "$target"
    ```

- To fix [this bug with yum and overlayfs](https://github.com/moby/moby/issues/10180) yum-plugin-ovl package needs to be installed.
Creating rootfs for our image:
    ```
    ./mkimage-yum.sh -p yum-plugin-ovl centos65
    ```

- Exit the container, we don't need it anymore (type exit or Ctrl+D)
    ```
    exit
    ```

- Import the contents from a tarball to create a filesystem image:
    ```
    docker import centos65.tar roman8422/centos6.5
    ```

- The only thing left is to verify the image:
    ```
    docker run --rm roman8422/centos6.5 cat /etc/issue
    CentOS release 6.5 (Final)
    Kernel \r on an \m
    ```
