FROM node:latest AS storefront
WORKDIR /usr/src/atsea/app/react-app
COPY react-app .
RUN npm install
RUN npm run build

FROM maven:3.6-openjdk-8 AS appserver
WORKDIR /usr/src/atsea
COPY pom.xml .
RUN mvn -B -f pom.xml -s /usr/share/maven/ref/settings-docker.xml dependency:resolve
COPY . .
RUN mvn -B -s /usr/share/maven/ref/settings-docker.xml package -DskipTests

# to get compiled nsjail binary
FROM nsjail AS jail

# ubuntu 18 instead of alpine
FROM java:8
COPY --from=jail /bin/nsjail /bin/nsjail

# to get ubuntu apt sources
COPY sources.list /etc/apt/sources.list

# no idea who put it in there but it contains sources for debian and we are working 
# on ubuntu so we get rid of it
RUN rm /etc/apt/sources.list.d/jessie-backports.list
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3B4FE6ACC0B21F32
RUN apt update -y

# libs required by nsjail
RUN apt install -y --force-yes strace autoconf bison flex gcc g++ git libprotobuf-dev libtool make pkg-config protobuf-compiler libprotobuf10 libnl-route-3-200

# to save some space
RUN rm -rf /var/lib/apt/lists/*
RUN useradd -ms /home/gordon gordon
WORKDIR /
COPY --from=storefront /usr/src/atsea/app/react-app/build/ /static
COPY --from=appserver /usr/src/atsea/target/AtSea-0.0.1-SNAPSHOT.jar /app/AtSea-0.0.1-SNAPSHOT.jar
COPY config.cfg /config.cfg

CMD ["nsjail", "--config", "config.cfg"]