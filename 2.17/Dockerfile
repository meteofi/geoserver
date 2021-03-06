ARG TOMCAT_VERSION="9"

FROM docker.io/tomcat:$TOMCAT_VERSION
LABEL maintainer "Mikko Rauhala <mikko@meteo.fi>"

ARG GEOSERVER_VERSION="2.17.5"

# persistent / runtime deps
RUN apt-get update && apt-get install -y --no-install-recommends libnetcdf-c++4 curl patch certbot && rm -r /var/lib/apt/lists/*

ENV NOTO_FONTS="NotoSans-unhinted NotoSerif-unhinted NotoMono-hinted" \
    GOOGLE_FONTS="Open%20Sans Roboto Lato Ubuntu" \
    GEOSERVER_VERSION=$GEOSERVER_VERSION \
    GEOSERVER_PLUGINS="css grib netcdf pyramid vectortiles wps ysld" \
    GEOSERVER_NODE_OPTS='id:$host_name' \
    GEOWEBCACHE_CACHE_DIR='/data/gwc' \
    JAVA_OPTS="-Xms128m -XX:SoftRefLRUPolicyMSPerMB=36000 -XX:+UseG1GC"


# Install Google Noto fonts
RUN mkdir -p /usr/share/fonts/truetype/noto && \
    for FONT in ${NOTO_FONTS}; \
    do \
        curl -sS -O https://noto-website-2.storage.googleapis.com/pkgs/${FONT}.zip && \
    	unzip -o ${FONT}.zip -d /usr/share/fonts/truetype/noto && \
    	rm -f ${FONT}.zip ; \
    done

# Install Google Fonts
RUN \
    for FONT in $GOOGLE_FONTS; \
    do \
        mkdir -p /usr/share/fonts/truetype/${FONT} && \
        curl -sS -o ${FONT}.zip "https://fonts.google.com/download?family=${FONT}" && \
    	unzip -o ${FONT}.zip -d /usr/share/fonts/truetype/${FONT} && \
    	rm -f ${FONT}.zip ; \
    done

COPY web.xml.patch .

# Install GeoServer
RUN curl -sS -L -O https://sourceforge.net/projects/geoserver/files/GeoServer/$GEOSERVER_VERSION/geoserver-$GEOSERVER_VERSION-war.zip && \
    unzip -d $CATALINA_HOME/webapps/ geoserver-$GEOSERVER_VERSION-war.zip geoserver.war && \
    unzip -d $CATALINA_HOME/webapps/geoserver $CATALINA_HOME/webapps/geoserver.war && \
    rm geoserver-$GEOSERVER_VERSION-war.zip $CATALINA_HOME/webapps/geoserver.war && \
    patch -i web.xml.patch $CATALINA_HOME/webapps/geoserver/WEB-INF/web.xml && \
    rm -rf $CATALINA_HOME/webapps/geoserver/data/workspaces/tiger $CATALINA_HOME/webapps/geoserver/data/workspaces/nurc $CATALINA_HOME/webapps/geoserver/data/workspaces/sf $CATALINA_HOME/webapps/geoserver/data/workspaces/topp $CATALINA_HOME/webapps/geoserver/data/layergroups/*

# Install GeoServer Plugins
RUN for PLUGIN in ${GEOSERVER_PLUGINS}; \
    do \
      curl -sS -L -O https://sourceforge.net/projects/geoserver/files/GeoServer/$GEOSERVER_VERSION/extensions/geoserver-$GEOSERVER_VERSION-$PLUGIN-plugin.zip && \
      unzip -o geoserver-$GEOSERVER_VERSION-$PLUGIN-plugin.zip -d $CATALINA_HOME/webapps/geoserver/WEB-INF/lib/ && \
      rm geoserver-$GEOSERVER_VERSION-$PLUGIN-plugin.zip ; \
    done

# Expose GeoServer's default port
EXPOSE 8080
EXPOSE 8443

HEALTHCHECK --interval=30s --timeout=10s\
    CMD curl -f "http://localhost:8080/geoserver/ows?service=wms&version=1.3.0&request=GetCapabilities" || exit 1

COPY docker-entrypoint.sh /
COPY server.xml $CATALINA_HOME/conf/
#COPY privkey.pem cert.pem $CATALINA_HOME/conf/

RUN mkdir -p $GEOWEBCACHE_CACHE_DIR && \
    chgrp -R 0 $CATALINA_HOME $GEOWEBCACHE_CACHE_DIR && \
    chmod -R g=u $CATALINA_HOME $GEOWEBCACHE_CACHE_DIR /etc/passwd /var/log

### Containers should NOT run as root as a good practice
USER 101010

ENTRYPOINT ["/docker-entrypoint.sh"]
CMD ["geoserver"]

