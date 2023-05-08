FROM ubuntu:20.04

# Update apt
ENV DEBIAN_FRONTEND="noninteractive"
RUN apt-get update -y

# Build Variables
ARG ANDROID_BUILD_TOOLS_VERSION="30.0.3"
ARG ANDROID_PLATFORM_VERSION="30"
ARG ANDROID_CMDLINE_TOOLS_BUILD="7583922"
ARG ANDROID_CMDLINE_TOOLS_VERSION="5.0"
ARG FLUTTER_VERSION="2.10.5"
ARG JAVA_VERSION="11"
ARG USER="developer"

# Set Timezone & Language
ENV ACCEPT_EULA="Y"
# ENV LANG="en_US.UTF-8"
# ENV LANGUAGE="en_US.UTF-8"
# ENV LC_ALL="en_US.UTF-8"
ENV TZ="America/Sao_Paulo"
ENV TIME_ZONE="$TZ"
RUN export
RUN echo "${TZ}" | tee /etc/timezone
RUN ln -fs "/usr/share/zoneinfo/${TZ}" /etc/localtime
# RUN dpkg-reconfigure -f noninteractive tzdata

# Install dependencies
# ENV DEBIAN_FRONTEND="noninteractive"
# RUN apt-get update -y
RUN apt-get install -fyq libxdamage1 libxtst6 libglu1-mesa
RUN apt-get install -fyq sudo curl unzip sed git bash
RUN apt-get install -fyq xz-utils libglvnd0 ssh xauth x11-xserver-utils libpulse0 libxcomposite1 libgl1-mesa-glx
RUN apt-get install -fyq openjdk-$JAVA_VERSION-jdk
RUN apt-get install -fyq chromium-browser
RUN apt-get install -fyq
RUN sudo rm -rf /var/lib/{apt,dpkg,cache,log}

# create user
ENV UID=1000
ENV GID=1000
ENV USER="$USER"
ENV HOME="/home/$USER"
RUN groupadd --gid $GID $USER
RUN useradd -s /bin/bash --uid $UID --gid $GID -m $USER
RUN echo $USER ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USER
RUN chmod 0440 /etc/sudoers.d/$USER

# Run Variables
ENV JAVA_VERSION="$JAVA_VERSION"
ENV ANDROID_CMDLINE_TOOLS_URL="https://dl.google.com/android/repository/commandlinetools-linux-${ANDROID_CMDLINE_TOOLS_BUILD}_latest.zip"
ENV ANDROID_PLATFORM_VERSION="$ANDROID_PLATFORM_VERSION"
ENV ANDROID_BUILD_TOOLS_VERSION="$ANDROID_BUILD_TOOLS_VERSION"
# ENV ANDROID_ARCHITECTURE="x86_64"
# RUN ANDROID_ARCHITECTURE=${ANDROID_ARCHITECTURE:-$(uname -m)}
ENV ANDROID_SDK_ROOT="$HOME/Android/sdk" ANDROID_HOME="$HOME/Android/sdk"
ENV ANDROID_CMDLINE_TOOLS_ROOT="$ANDROID_HOME/cmdline-tools"
ENV ANDROID_CMDLINE_TOOLS_HOME="$ANDROID_CMDLINE_TOOLS_ROOT/$ANDROID_CMDLINE_TOOLS_VERSION"
ENV ANDROID_SDK_HOME="$HOME/.android" ANDROID_USER_HOME="$HOME/.android"
ENV FLUTTER_CHANNEL="stable"
ENV FLUTTER_VERSION="$FLUTTER_VERSION"
ENV FLUTTER_URL="https://storage.googleapis.com/flutter_infra_release/releases/$FLUTTER_CHANNEL/linux/flutter_linux_$FLUTTER_VERSION-$FLUTTER_CHANNEL.tar.xz"
ENV FLUTTER_HOME="$HOME/flutter"
ENV FLUTTER_WEB_PORT="8090"
ENV FLUTTER_DEBUG_PORT="42000"
ENV FLUTTER_EMULATOR_NAME="flutter_emulator"

USER $USER
WORKDIR $HOME

# RUN printenv
# RUN export

# android sdk
ENV PATH="$ANDROID_CMDLINE_TOOLS_HOME/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/platforms:$ANDROID_HOME/emulator:$PATH"
RUN mkdir -p "$ANDROID_USER_HOME"
RUN touch "$ANDROID_USER_HOME/repositories.cfg"
RUN mkdir -p "$ANDROID_HOME"
RUN mkdir -p "$ANDROID_CMDLINE_TOOLS_ROOT"
RUN curl -s "$ANDROID_CMDLINE_TOOLS_URL" -o 'commandlinetools-linux--latest.zip'
RUN unzip -qq 'commandlinetools-linux--latest.zip'
# RUN mv "cmdline-tools" "${ANDROID_CMDLINE_TOOLS_HOME}"
RUN rm 'commandlinetools-linux--latest.zip'
RUN yes "y" | ./cmdline-tools/bin/sdkmanager --sdk_root="${ANDROID_HOME}" --install "cmdline-tools;${ANDROID_CMDLINE_TOOLS_VERSION}" > /dev/null
RUN rm -rf cmdline-tools
RUN yes "y" | sdkmanager --licenses > /dev/null
RUN sdkmanager --version
RUN sdkmanager --list_installed --include_obsolete
RUN sdkmanager --install "tools" > /dev/null
RUN sdkmanager --install "build-tools;$ANDROID_BUILD_TOOLS_VERSION" > /dev/null
RUN sdkmanager --install "platforms;android-$ANDROID_PLATFORM_VERSION" > /dev/null
RUN sdkmanager --install "platform-tools" > /dev/null
RUN sdkmanager --install "emulator" > /dev/null
RUN sdkmanager --install "system-images;android-$ANDROID_PLATFORM_VERSION;google_apis_playstore;${ANDROID_ARCHITECTURE:-$(uname -m)}" > /dev/null
RUN sdkmanager --update > /dev/null
RUN sdkmanager --list_installed --include_obsolete
RUN sdkmanager --licenses > /dev/null

# flutter
ENV PATH="$FLUTTER_HOME/bin:$PATH"
RUN curl -s $FLUTTER_URL -o flutter.tar.xz
RUN mkdir -p $FLUTTER_HOME
RUN tar xf flutter.tar.xz -C $HOME
RUN rm -f flutter.tar.xz
RUN flutter config --android-sdk="${ANDROID_HOME}"
RUN flutter config --enable-android
RUN flutter precache
RUN yes "y" | flutter doctor --android-licenses
RUN flutter doctor
RUN flutter emulators --create
RUN flutter update-packages

COPY --chmod=555 entrypoint.sh /usr/local/bin/
COPY --chmod=555 chown.sh /usr/local/bin/
COPY --chmod=555 flutter-android-emulator.sh /usr/local/bin/flutter-android-emulator
COPY --chmod=555 flutter-web.sh /usr/local/bin/flutter-web
# RUN chmod +x /usr/local/bin/entrypoint.sh
# RUN chmod +x /usr/local/bin/chown.sh
# RUN chmod +x /usr/local/bin/flutter-android-emulator
# RUN chmod +x /usr/local/bin/flutter-web
ENTRYPOINT [ "/usr/local/bin/entrypoint.sh" ]
