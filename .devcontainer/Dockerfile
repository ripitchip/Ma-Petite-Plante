# 1. Base Image
FROM mcr.microsoft.com/devcontainers/base:debian-12

# 2. Set Environment Variables
ENV SHELL=/bin/zsh
ENV ANDROID_HOME=/home/vscode/Android/Sdk
ENV NDK_HOME=${ANDROID_HOME}/ndk/26.1.10909125
ENV JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
# Ajout du chemin de l'émulateur au PATH
ENV PATH="${PATH}:${ANDROID_HOME}/cmdline-tools/latest/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/emulator:/home/vscode/.cargo/bin:${JAVA_HOME}/bin"

# 3. Install System Dependencies & Fix GPG/Certificates
RUN apt-get update && apt-get install -y debian-archive-keyring \
    && apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get install -y --no-install-recommends \
        pkg-config libglib2.0-dev libgtk-3-dev libwebkit2gtk-4.1-dev \
        libayatana-appindicator3-dev librsvg2-dev libssl-dev curl wget \
        build-essential python3-full python3-dev python3-pip \
        libpython3.11-dev x11-apps mesa-utils sudo xdg-utils \
        libfuse2 file appstream unzip ca-certificates \
        openjdk-17-jdk-headless \
        libpulse0 libgluegen2-rt-java \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# 4. Install Node.js & pnpm
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_20.x nodistro main" | tee /etc/apt/sources.list.d/nodesource.list \
    && apt-get update && apt-get install -y nodejs \
    && npm install -g pnpm

# 5. Setup Python Tools
RUN python3 -m pip install --upgrade pip --break-system-packages \
    && python3 -m pip install pyinstaller google-api-python-client google-auth-oauthlib --break-system-packages

# 6. Setup Android SDK, NDK & Emulator
USER vscode
RUN mkdir -p $ANDROID_HOME/cmdline-tools \
    && wget https://dl.google.com/android/repository/commandlinetools-linux-11076708_latest.zip -O /tmp/cmdline.zip \
    && unzip /tmp/cmdline.zip -d $ANDROID_HOME/cmdline-tools \
    && mv $ANDROID_HOME/cmdline-tools/cmdline-tools $ANDROID_HOME/cmdline-tools/latest \
    && rm /tmp/cmdline.zip

# Installation du SDK, NDK, Emulator et Image Système
RUN yes | $ANDROID_HOME/cmdline-tools/latest/bin/sdkmanager --sdk_root=$ANDROID_HOME \
    "platform-tools" \
    "platforms;android-34" \
    "build-tools;34.0.0" \
    "ndk;26.1.10909125" \
    "emulator" \
    "system-images;android-34;google_apis;x86_64"

# Création de l'AVD (Émulateur)
RUN echo "no" | $ANDROID_HOME/cmdline-tools/latest/bin/avdmanager create avd \
    -n tauri_emu \
    -k "system-images;android-34;google_apis;x86_64"

# 7. Install Rust & Targets
RUN curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y \
    && /home/vscode/.cargo/bin/rustup target add aarch64-linux-android armv7-linux-androideabi i686-linux-android x86_64-linux-android

# 8. Setup Zsh Plugins
RUN git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting \
    && git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions \
    && sed -i 's/plugins=(git)/plugins=(git zsh-syntax-highlighting zsh-autosuggestions)/' ~/.zshrc

# 9. Final touches
USER root
RUN mkdir -p /run/user/1000 && chown -R vscode:vscode /run/user/1000 && chmod 700 /run/user/1000

RUN echo '#!/bin/bash\n\
/usr/bin/python3 -m PyInstaller --onefile --noconsole --clean src-python/main.py\n\
TRIPLE=$(rustc -Vv | grep host | cut -d " " -f 2)\n\
if [ -f "dist/main" ]; then\n\
    mkdir -p src-tauri/binaries\n\
    mv dist/main "src-tauri/binaries/backend-$TRIPLE"\n\
    echo "Done: backend-$TRIPLE created."\n\
else\n\
    echo "Error: dist/main not found."\n\
fi' > /usr/local/bin/build-backend && chmod +x /usr/local/bin/build-backend

USER vscode