FROM ubuntu:20.04

# Android SDK Environment variables
# See: https://developer.android.com/studio#downloads
ENV ANDROID_SDK_TOOLS_VERSION 8092744
ENV ANDROID_PLATFORM_VERSION 29
ENV ANDROID_BUILD_TOOLS_VERSION 29.0.3
ENV ANDROID_HOME=/opt/android-sdk-linux
ENV ANDROID_SDK_ROOT="$ANDROID_HOME"
ENV PATH=${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator

# Flutter SDK
# https://flutter.dev/docs/development/tools/sdk/releases?tab=linux
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_VERSION="2.10.0"
ENV FLUTTER_HOME=/opt/flutter
ENV PATH=${PATH}:${FLUTTER_HOME}/bin

# This Dockerfile adds a non-root user with sudo access. Use the "remoteUser"
# property in devcontainer.json to use it. On Linux, the container user's GID/UIDs
# will be updated to match your local UID/GID (when using the dockerFile property).
# See https://aka.ms/vscode-remote/containers/non-root-user for details.
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# The container is configured for non-root user but we need to access "/dev/kvm" to run emulators.
# We have to map the group owner of "/dev/kvm` from host (which is "kvm") to inside the container.
# So we'll create a new "kvm" group with the id from host returned in "cat /etc/group | grep kvm"). 
ARG KVM_GID=108

# Install needed packages, setup user and clean up.
RUN apt-get update \
	&& apt-get update \
		&& DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
		sudo \
        openjdk-8-jdk-headless \
	    wget curl git xz-utils zip unzip libglu1 \
        libpulse-dev libxcomposite-dev libxcursor-dev libasound2-dev kmod qemu-kvm \
        ssh vim locate \
	# Clean Up
	&& apt-get autoremove -y \
	&& apt-get clean -y \
	&& rm -rf /var/lib/apt/lists/* \
	# Create a non-root user to use if preferred - see https://aka.ms/vscode-remote/containers/non-root-user.
	# [Optional] Add sudo support for the non-root user
	&& groupadd --gid $USER_GID $USERNAME \
	&& useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME \
	&& echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
	&& chmod 0440 /etc/sudoers.d/$USERNAME \
	&& groupadd -g $KVM_GID kvm \
    && adduser $USERNAME kvm

# Install
# Android SDK	
RUN cd /opt \
	&& curl -C - --output android-sdk-tools.zip https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_SDK_TOOLS_VERSION}_latest.zip \
	&& mkdir -p ${ANDROID_HOME}/cmdline-tools \
	&& unzip -q android-sdk-tools.zip -d ${ANDROID_HOME}/cmdline-tools \
	&& mv ${ANDROID_HOME}/cmdline-tools/cmdline-tools ${ANDROID_HOME}/cmdline-tools/latest  \
	&& rm android-sdk-tools.zip \
	&& yes | sdkmanager --licenses \
	&& touch $HOME/.android/repositories.cfg \
	&& sdkmanager platform-tools \
	&& sdkmanager emulator \
	&& sdkmanager "platforms;android-$ANDROID_PLATFORM_VERSION" "build-tools;$ANDROID_BUILD_TOOLS_VERSION" \
	&& sdkmanager "system-images;android-27;google_apis_playstore;x86"

# Install
# Flutter SDK
RUN cd /opt \
	&& curl -C - --output flutter.tar.xz https://storage.googleapis.com/flutter_infra_release/releases/${FLUTTER_CHANNEL}/linux/flutter_linux_${FLUTTER_VERSION}-${FLUTTER_CHANNEL}.tar.xz \
	&& tar xf flutter.tar.xz -C . \
	&& rm flutter.tar.xz \
	&& yes | flutter doctor --android-licenses \
	&& flutter config --no-analytics \
	&& flutter config --enable-web \
	&& flutter update-packages \
	&& flutter emulators --create --name android-emulator
