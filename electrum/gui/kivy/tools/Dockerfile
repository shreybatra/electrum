# based on https://github.com/kivy/python-for-android/blob/master/Dockerfile

FROM ubuntu:18.04

ENV ANDROID_HOME="/opt/android"

RUN apt -y update -qq \
    && apt -y install -qq --no-install-recommends curl unzip git python3-pip python3-setuptools \
    && apt -y autoremove \
    && apt -y clean


ENV ANDROID_NDK_HOME="${ANDROID_HOME}/android-ndk"
ENV ANDROID_NDK_VERSION="14b"
ENV ANDROID_NDK_HOME_V="${ANDROID_NDK_HOME}-r${ANDROID_NDK_VERSION}"

# get the latest version from https://developer.android.com/ndk/downloads/index.html
ENV ANDROID_NDK_ARCHIVE="android-ndk-r${ANDROID_NDK_VERSION}-linux-x86_64.zip"
ENV ANDROID_NDK_DL_URL="https://dl.google.com/android/repository/${ANDROID_NDK_ARCHIVE}"

# download and install Android NDK
RUN curl --location --progress-bar \
        "${ANDROID_NDK_DL_URL}" \
        --output "${ANDROID_NDK_ARCHIVE}" \
    && mkdir --parents "${ANDROID_NDK_HOME_V}" \
    && unzip -q "${ANDROID_NDK_ARCHIVE}" -d "${ANDROID_HOME}" \
    && ln -sfn "${ANDROID_NDK_HOME_V}" "${ANDROID_NDK_HOME}" \
    && rm -rf "${ANDROID_NDK_ARCHIVE}"


ENV ANDROID_SDK_HOME="${ANDROID_HOME}/android-sdk"

# get the latest version from https://developer.android.com/studio/index.html
ENV ANDROID_SDK_TOOLS_VERSION="4333796"
ENV ANDROID_SDK_TOOLS_ARCHIVE="sdk-tools-linux-${ANDROID_SDK_TOOLS_VERSION}.zip"
ENV ANDROID_SDK_TOOLS_DL_URL="https://dl.google.com/android/repository/${ANDROID_SDK_TOOLS_ARCHIVE}"

# download and install Android SDK
RUN curl --location --progress-bar \
        "${ANDROID_SDK_TOOLS_DL_URL}" \
        --output "${ANDROID_SDK_TOOLS_ARCHIVE}" \
    && mkdir --parents "${ANDROID_SDK_HOME}" \
    && unzip -q "${ANDROID_SDK_TOOLS_ARCHIVE}" -d "${ANDROID_SDK_HOME}" \
    && rm -rf "${ANDROID_SDK_TOOLS_ARCHIVE}"

# update Android SDK, install Android API, Build Tools...
RUN mkdir --parents "${ANDROID_SDK_HOME}/.android/" \
    && echo '### User Sources for Android SDK Manager' \
        > "${ANDROID_SDK_HOME}/.android/repositories.cfg"

# accept Android licenses (JDK necessary!)
RUN apt -y update -qq \
    && apt -y install -qq --no-install-recommends openjdk-8-jdk \
    && apt -y autoremove \
    && apt -y clean
RUN yes | "${ANDROID_SDK_HOME}/tools/bin/sdkmanager" --licenses > /dev/null

# download platforms, API, build tools
RUN "${ANDROID_SDK_HOME}/tools/bin/sdkmanager" "platforms;android-24" > /dev/null && \
    "${ANDROID_SDK_HOME}/tools/bin/sdkmanager" "platforms;android-28" > /dev/null && \
    "${ANDROID_SDK_HOME}/tools/bin/sdkmanager" "build-tools;28.0.3" > /dev/null && \
    "${ANDROID_SDK_HOME}/tools/bin/sdkmanager" "extras;android;m2repository" > /dev/null && \
    chmod +x "${ANDROID_SDK_HOME}/tools/bin/avdmanager"


ENV USER="user"
ENV HOME_DIR="/home/${USER}"
ENV WORK_DIR="${HOME_DIR}/wspace" \
    PATH="${HOME_DIR}/.local/bin:${PATH}"

# install system dependencies
RUN apt -y update -qq \
    && apt -y install -qq --no-install-recommends \
        python virtualenv python-pip wget lbzip2 patch sudo \
        software-properties-common

# install kivy
RUN add-apt-repository ppa:kivy-team/kivy \
    && apt -y update -qq  \
    && apt -y install -qq --no-install-recommends python3-kivy \
    && apt -y autoremove \
    && apt -y clean
RUN python3 -m pip install image

# build dependencies
# https://buildozer.readthedocs.io/en/latest/installation.html#android-on-ubuntu-16-04-64bit
RUN dpkg --add-architecture i386 \
    && apt -y update -qq \
    && apt -y install -qq --no-install-recommends \
        build-essential ccache git python2.7 python2.7-dev \
        libncurses5:i386 libstdc++6:i386 libgtk2.0-0:i386 \
        libpangox-1.0-0:i386 libpangoxft-1.0-0:i386 libidn11:i386 \
        zip zlib1g-dev zlib1g:i386 \
    && apt -y autoremove \
    && apt -y clean

# specific recipes dependencies (e.g. libffi requires autoreconf binary)
RUN apt -y update -qq \
    && apt -y install -qq --no-install-recommends \
        autoconf automake cmake gettext libltdl-dev libtool pkg-config \
        python3.7 \
    && apt -y autoremove \
    && apt -y clean


# prepare non root env
RUN useradd --create-home --shell /bin/bash ${USER}

# with sudo access and no password
RUN usermod -append --groups sudo ${USER}
RUN echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers


WORKDIR ${WORK_DIR}

# user needs ownership/write access to these directories
RUN chown --recursive ${USER} ${WORK_DIR} ${ANDROID_SDK_HOME}
RUN chown ${USER} /opt
USER ${USER}


RUN pip install --upgrade cython==0.29
RUN python3 -m pip install --upgrade cython==0.29

# install buildozer
RUN cd /opt \
    && git clone https://github.com/kivy/buildozer \
    && cd buildozer \
    && python3 -m pip install -e .

# install python-for-android
RUN cd /opt \
    && git clone https://github.com/kivy/python-for-android \
    && cd python-for-android \
    && git remote add sombernight https://github.com/SomberNight/python-for-android \
    && git fetch --all \
    && git checkout f74226666af69f9915afaee9ef9292db85a6c617 \
    && python3 -m pip install -e .

# build env vars
ENV USE_SDK_WRAPPER=1
ENV GRADLE_OPTS="-Xmx1536M -Dorg.gradle.jvmargs='-Xmx1536M'"
