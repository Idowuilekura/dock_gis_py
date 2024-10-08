FROM python:3.9-slim-bookworm AS builder

WORKDIR /app


# ENV PYTHONDONTWRITEBYTECODE=1
# ENV PYTHONUNBUFFERED=1
# --no-install-recommends

RUN apt update && apt install --no-install-recommends -y curl ca-certificates g++ gcc openmpi-bin libgdal-dev libopenmpi-dev libboost-iostreams-dev python3-dev git
# sudo apt install openmpi-bin libgdal-dev libopenmpi-dev libboost-iostreams-dev
# g++ python3-dev gcc

# ENV PROJ_DIR=/usr/local/bin/proj

ADD https://astral.sh/uv/install.sh /uv-installer.sh

RUN sh /uv-installer.sh && rm /uv-installer.sh

ENV PATH="/root/.cargo/bin/:$PATH"

RUN uv venv /opt/venv

# RUN uv venv  /app/venv
ENV VIRTUAL_ENV=/opt/venv

ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt /opt/necessary/requirements.txt

# COPY requirements.txt .
# RUN uv pip install Cython 
RUN uv pip install richdem
# RUN git clone --recursive https://github.com/r-barnes/richdem.git ghrichdem
# RUN cd ghrichdem/wrappers/pyrichdem && uv pip install .
RUN uv pip install --strict --no-cache -r /opt/necessary/requirements.txt 
RUN uv cache clean 



# final stage
FROM python:3.9-slim-bookworm AS final

RUN apt update && apt install -y python3-launchpadlib wget 
COPY --from=builder /opt/venv /opt/venv
COPY --from=builder /root/.cargo/bin /root/.cargo/bin
# COPY --from=builder /opt/venv/bin /opt/venv/bin
# COPY --from=builder /opt/venv/lib /opt/venv/lib
# COPY --from=builder /opt/venv/pyvenv.cfg /opt/venv/pyvenv.cfg
# COPY --from=builder /bin/sh /bin/sh
RUN apt-get update --fix-missing \
    && apt-get install -y software-properties-common \
    && add-apt-repository ppa:ubuntugis/ubuntugis-unstable \
    && apt-get update && \
    apt-get clean && \
    apt update && apt install  -y --no-install-recommends \ 
    gdal-bin libgdal-dev gcc build-essential libtiff5-dev && \
    rm -rf /var/lib/apt/lists/* && \
    rm -r /root/.cache && rm -rf /var/cache/apt/*
    
    ENV CPLUS_INCLUDE_PATH=/usr/include/gdal
    ENV C_INCLUDE_PATH=/usr/include/gdal

ENV PATH="/root/.cargo/bin/:$PATH"
ENV VIRTUAL_ENV="/opt/venv"

ENV PATH="/opt/venv/bin:$PATH"

RUN uv pip install GDAL==`gdal-config --version`

# COPY ./install_spark.sh /opt/necessary/install_spark.sh

# RUN chmod +x /opt/necessary/install_spark.sh

# RUN sh /opt/necessary/install_spark.sh
USER root 
RUN mkdir -p /opt/spark_home
ENV SPARK_HOME=/opt/spark_home
# export SPARK_HOME="${SPARK_HOME}"
RUN wget https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_linux-x64_bin.tar.gz && tar -xvf openjdk-11.0.2_linux-x64_bin.tar.gz  --directory ${SPARK_HOME}
RUN rm -f openjdk-11.0.2_linux-x64_bin.tar.gz 
ENV JAVA_HOME="${SPARK_HOME}/jdk-11.0.2"
ENV PATH="${JAVA_HOME}/bin:${PATH}"
RUN wget https://archive.apache.org/dist/spark/spark-3.3.2/spark-3.3.2-bin-hadoop3.tgz && tar -xvf spark-3.3.2-bin-hadoop3.tgz --directory ${SPARK_HOME}
RUN rm -f spark-3.3.2-bin-hadoop3.tgz
ENV SPARK_HOME="${SPARK_HOME}/spark-3.3.2-bin-hadoop3"
ENV PATH="${SPARK_HOME}/bin:${PATH}" 
ENV PYTHONPATH="${SPARK_HOME}/python/:$PYTHONPATH"
ENV PYTHONPATH="${SPARK_HOME}/python/lib/py4j-0.10.9.5-src.zip:$PYTHONPATH"

RUN uv pip install pyspark


WORKDIR /app


COPY entrypoint.sh /opt/necessary/entrypoint.sh
# COPY source_file.sh /app/source_file.sh



RUN chmod u+x /opt/necessary/entrypoint.sh

EXPOSE 8888


CMD ["/opt/necessary/entrypoint.sh"]
