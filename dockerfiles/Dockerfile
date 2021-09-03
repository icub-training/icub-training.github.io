FROM ubuntu:latest
LABEL org.opencontainers.image.title="iCub Training Sandbox"
LABEL org.opencontainers.image.description="Infrastructure for iCub Training"
LABEL org.opencontainers.image.source="https://github.com/icub-training/icub-training.github.io"
LABEL org.opencontainers.image.authors="Ugo Pattacini <ugo.pattacini@iit.it>"

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Update apt database
RUN apt update

# Install essentials
RUN apt install -y apt-utils software-properties-common apt-transport-https sudo \
    psmisc tmux nano wget curl gedit gdb git gitk autoconf locales gdebi terminator \
    meld dos2unix

# Set the locale
RUN locale-gen en_US.UTF-8

# Install VSCode
# better off downling the deb file than relying on packages.microsoft.com sources that are often broken
RUN wget -O code.deb https://go.microsoft.com/fwlink/?LinkID=760868 && \
    gdebi -n code.deb && \
    rm code.deb

# Install graphics
RUN apt install -y xfce4 xfce4-goodies xserver-xorg-video-dummy xserver-xorg-legacy x11vnc firefox && \
    apt remove -y xfce4-power-manager light-locker && \
    sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config
COPY xorg.conf /etc/X11/xorg.conf
RUN dos2unix /etc/X11/xorg.conf

# Install Octave
RUN apt install -y octave epstool transfig

# Install python
RUN apt install -y python3 python3-dev python3-pip python3-setuptools && \
    if [ ! -f "/usr/bin/python" ]; then ln -s /usr/bin/python3 /usr/bin/python; fi
    
# Install magic-wormwhole to get things from one computer to another safely
RUN apt install -y magic-wormhole

# Install noVNC
RUN git clone https://github.com/novnc/noVNC.git /opt/novnc && \
    git clone https://github.com/novnc/websockify /opt/novnc/utils/websockify && \
    echo "<html><head><meta http-equiv=\"Refresh\" content=\"0; url=vnc.html?autoconnect=true&reconnect=true&reconnect_delay=1000&resize=scale&quality=9\"></head></html>" > /opt/novnc/index.html

# Select options
ARG ROBOTOLOGY_SUPERBUILD_RELEASE
ARG BUILD_TYPE
ARG ROBOTOLOGY_SUPERBUILD_INSTALL_DIR=/usr/local

# Set up git (required by superbuild)
RUN git config --global user.name "GitHub Actions" && \
    git config --global user.email "actions@github.com"

# Install dependencies
RUN git clone https://github.com/robotology/robotology-superbuild.git --depth 1 --branch ${ROBOTOLOGY_SUPERBUILD_RELEASE} && \
    sudo robotology-superbuild/scripts/install_apt_dependencies.sh

# Clean up git configuration
RUN git config --global --unset-all user.name && \
    git config --global --unset-all user.email

# Install dependencies
RUN sh -c 'echo "deb http://packages.osrfoundation.org/gazebo/ubuntu-stable `lsb_release -cs` main" > /etc/apt/sources.list.d/gazebo-stable.list' && \
    wget https://packages.osrfoundation.org/gazebo.key -O - | apt-key add - && \
    apt update && \
    apt install -y gazebo11 libgazebo11-dev
    
# Build robotology-superbuild
RUN cd robotology-superbuild && \
    mkdir build && cd build && \
    cmake .. \
          -DCMAKE_BUILD_TYPE=${BUILD_TYPE} \
          -DYCM_EP_INSTALL_DIR=${ROBOTOLOGY_SUPERBUILD_INSTALL_DIR} \
          -DROBOTOLOGY_ENABLE_CORE:BOOL=ON \
          -DROBOTOLOGY_ENABLE_ROBOT_TESTING:BOOL=ON \
          -DROBOTOLOGY_USES_GAZEBO:BOOL=ON && \
    make && \
    cd ../.. && rm -Rf robotology-superbuild
    
# Set environmental variables
ENV DISPLAY=:1
ENV DONT_PROMPT_WSL_INSTALL=1

# Install informative git for bash
RUN git clone https://github.com/magicmonty/bash-git-prompt.git /root/.bash-git-prompt --depth=1

# Set up .bashrc
RUN echo "alias code='code --user-data-dir=\"/root/.vscode\"'" >> /root/.bashrc && \
    echo "source /etc/profile.d/bash_completion.sh" >> /root/.bashrc && \
    echo "GIT_PROMPT_ONLY_IN_REPO=1" >> /root/.bashrc && \
    echo "source \${HOME}/.bash-git-prompt/gitprompt.sh" >> /root/.bashrc && \
    echo "source ${ROBOTOLOGY_SUPERBUILD_INSTALL_DIR}/share/robotology-superbuild/setup.sh" >> /root/.bashrc

# Set up script to launch graphics and vnc
COPY start-vnc-session.sh /usr/bin/start-vnc-session.sh
RUN chmod +x /usr/bin/start-vnc-session.sh && \
    dos2unix /usr/bin/start-vnc-session.sh

# Set up VSCode launcher
COPY ["Visual Studio Code.desktop", "/root/Desktop/Visual Studio Code.desktop"]
RUN chmod +x "/root/Desktop/Visual Studio Code.desktop" && \
    dos2unix "/root/Desktop/Visual Studio Code.desktop"

# Manage ports
EXPOSE 5901 6080 10000/tcp 10000/udp

# Clean up unnecessary installation products
RUN rm -Rf /var/lib/apt/lists/*

# Launch bash from /root
WORKDIR /root
CMD ["bash"]
