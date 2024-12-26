FROM ubuntu:20.04


### Installation of softwares in the Dockerfile and cleanup.
# Update package lists and install required packages
RUN apt-get update && \
    apt-get install -y wget software-properties-common gnupg2 winbind xvfb curl


# Add Wine repository and install Wine
RUN dpkg --add-architecture i386 && \
    wget -nc https://dl.winehq.org/wine-builds/winehq.key && \
    apt-key add winehq.key && \
    add-apt-repository 'deb https://dl.winehq.org/wine-builds/ubuntu/ focal main' && \
    apt-get update && \
    apt-get install -y --install-recommends winehq-stable wine32


# Install additional packages and configure Wine
RUN apt-get install -y winetricks && \
    winetricks msxml6


# Install Python 3.8 and necessary tools
RUN apt-get update && \
    apt-get install -y python3.8 python3.8-venv python3.8-dev && \
    apt-get install -y python3-pip && \
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.8 1


# Install boto3 using pip
RUN pip3 install boto3 \
    && pip3 install plotly


# Install AWS Lambda Runtime Interface Client (awslambdaric)
RUN pip3 install awslambdaric


# RUN Download custom AWS Lambda runtime interface emulator from github
RUN curl -Lo /usr/local/bin/aws-lambda-rie https://github.com/MecSimCalc/aws-lambda-runtime-interface-emulator/raw/msc-v1.13/bin/aws-lambda-rie && \
    chmod +x /usr/local/bin/aws-lambda-rie


# Cleanup unnecessary files
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*


### Clean logs for wine, allow applications with GUI terminal dialog boxes to run with Wine, and set directories for the web server.
# Set the Wine debug level to suppress non-essential messages 
ENV WINEDEBUG=fixme-all


# Allow applications with GUI terminal dialog boxes to run with Wine
ENV DISPLAY=:99
RUN Xvfb :99 -screen 0 1024x768x16 &


# Set LAMBDA_TASK_ROOT environment variable
ENV LAMBDA_TASK_ROOT=/var/task


# Set LAMBDA_RUNTIME_DIR environment variable
ENV LAMBDA_RUNTIME_DIR=/var/runtime


### Copy files and folder from host machine to Docker image, and turn lambda-entrypoint.sh into an executable.
# COPY Files from local build folder
COPY Project1.exe ${LAMBDA_TASK_ROOT}/
COPY download_and_run.py ${LAMBDA_TASK_ROOT}/
COPY app.py ${LAMBDA_TASK_ROOT}/
COPY lambda-entrypoint.sh /


# Create /var directory and subdirectories and copy the contents of var
RUN mkdir -p /var/adm \
    && mkdir -p /var/cache \
    && mkdir -p /var/db \
    && mkdir -p /var/empty \
    && mkdir -p /var/games \
    && mkdir -p /var/gopher \
    && mkdir -p /var/kerberos \
    && mkdir -p /var/lang \
    && mkdir -p /var/lib \
    && mkdir -p /var/local \
    && mkdir -p /var/log \
    && mkdir -p /var/nis \
    && mkdir -p /var/opt \
    && mkdir -p /var/preserve \
    && mkdir -p /var/rapid \
    && mkdir -p /var/runtime \
    && mkdir -p /var/spool \
    && mkdir -p /var/task \
    && mkdir -p /var/telemetry \
    && mkdir -p /var/tmp \
    && mkdir -p /var/tracer \
    && mkdir -p /var/yp


COPY var/runtime/* /var/runtime/
COPY var/lang/ /var/lang/
COPY var/* /var/




# Ensure the entrypoint script is executable
RUN chmod +x /lambda-entrypoint.sh


### Set entrypoint and command to run the web server.
# Set the entrypoint to the script
ENTRYPOINT ["/lambda-entrypoint.sh"]


# Pass the handler function as an argument
CMD ["app.handler"]


### Set temporary directory as the working directory, and set API Access Key to MecSimCalc’s API key.
# WORKDIR as /tmp, which is the only writable directory
WORKDIR /tmp


# Environment variables set to the temporary directory
ENV MPLCONFIGDIR=/tmp
ENV TMPDIR=/tmp


# - API_ACCESS_KEY is the SECRET key used to access the Mecsimcalc private API
ARG API_ACCESS_KEY
ENV API_ACCESS_KEY=$API_ACCESS_KEY
