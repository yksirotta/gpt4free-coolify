FROM selenium/node-chrome

ARG G4F_VERSION
ENV G4F_VERSION=$G4F_VERSION

ENV SE_SCREEN_WIDTH=1850
ENV G4F_DIR=/g4f
ENV VIRTUAL_ENV=/opt/venv         
ENV G4F_LOGIN_URL=http://g4f.yksirotta.fi:7900/?autoconnect=1&resize=scale&password=secret
ENV PATH="$HOME/.local/bin:$PATH"
ENV PATH="$VIRTUAL_ENV/bin:$PATH"  

USER root

# If docker compose, install git
RUN if [ "$G4F_VERSION" = "" ]; then \
    apt-get -qqy update && \
    apt-get -qqy install git; \
fi

# Install Python3, pip, remove OpenJDK 11, clean up
RUN apt-get -qqy update && \
    apt-get -qqy upgrade && \
    apt-get -qyy autoremove && \
   apt-get -qqy install python3 python3-pip python3-venv build-essential g++ python3-dev && \
    apt-get -qyy remove openjdk-11-jre-headless && \
    apt-get -qyy autoremove && \
    apt-get -qyy clean && \
    rm -rf /var/lib/apt/lists/* /var/cache/apt/*

# Create and fix permissions for the virtual environment
ENV VIRTUAL_ENV=/opt/venv
RUN python3 -m venv $VIRTUAL_ENV
RUN chown -R $SEL_UID:$SEL_GID $VIRTUAL_ENV

# Upgrade pip and install spaCy in the virtual environment
RUN $VIRTUAL_ENV/bin/pip install --upgrade pip && \
    $VIRTUAL_ENV/bin/pip install --no-cache-dir spacy 

# Install the spaCy model directly
RUN $VIRTUAL_ENV/bin/python -m spacy download en_core_web_sm

# Update entrypoint
COPY docker/supervisor.conf /etc/supervisor/conf.d/selenium.conf
COPY docker/supervisor-api.conf /etc/supervisor/conf.d/api.conf

# Change background image
COPY docker/background.png /usr/share/images/fluxbox/ubuntu-light.png

# Add user, fix permissions
RUN chown "${SEL_UID}:${SEL_GID}" $HOME/.local

# Create the pip cache directory and change ownership
RUN mkdir -p /home/seluser/.cache/pip && \
    chown -R $SEL_UID:$SEL_GID /home/seluser/.cache/pip

# Switch user
USER $SEL_UID

# Set the working directory in the container.
WORKDIR $G4F_DIR

# Copy the project's requirements file into the container.
COPY requirements.txt $G4F_DIR

# Upgrade pip for the latest features and install the project's Python dependencies.
RUN pip install --break-system-packages --upgrade pip && \
    pip install --break-system-packages -r requirements.txt

# Copy the entire package into the container.
ADD --chown=$SEL_UID:$SEL_GID g4f $G4F_DIR/g4f

# Ensure the necessary directories exist and set the correct permissions
RUN mkdir -p $G4F_DIR/har_and_cookies $G4F_DIR/generated_images && \
    touch $G4F_DIR/har_and_cookies/.scrape_cache && \
    chmod -R 755 $G4F_DIR/har_and_cookies $G4F_DIR/generated_images && \
    chown -R $SEL_UID:$SEL_GID $G4F_DIR/har_and_cookies $G4F_DIR/generated_images


# Expose ports
EXPOSE 10555 7900 1337
