FROM microsoft/dotnet:2.0.5-sdk-2.1.4-stretch

# set up environment
ENV ASPNETCORE_URLS http://+:80
ENV NETCORE_1_0_RUNTIME_VERSION 1.0.9
ENV NETCORE_1_1_RUNTIME_VERSION 1.1.6
ENV ASPNETCORE_PKG_VERSION 2.0.5

# set up node
ENV NODE_VERSION 6.12.3
ENV NODE_DOWNLOAD_URL https://nodejs.org/dist/v$NODE_VERSION/node-v$NODE_VERSION-linux-x64.tar.gz
ENV NODE_DOWNLOAD_SHA 0F8144C84C4379CB35AE409779C062A65680CF163B52C4660932EB58CFA1D065

RUN curl -SL "$NODE_DOWNLOAD_URL" --output nodejs.tar.gz \
    && echo "$NODE_DOWNLOAD_SHA nodejs.tar.gz" | sha256sum -c - \
    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \
    && rm nodejs.tar.gz \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    # set up bower and gulp
    && npm install -g bower gulp \
    && echo '{ "allow_root": true }' > /root/.bowerrc

# Install the 1.x runtimes
RUN for url in \
      "https://dotnetcli.blob.core.windows.net/dotnet/Runtime/${NETCORE_1_0_RUNTIME_VERSION}/dotnet-debian-x64.${NETCORE_1_0_RUNTIME_VERSION}.tar.gz" \
      "https://dotnetcli.blob.core.windows.net/dotnet/Runtime/${NETCORE_1_1_RUNTIME_VERSION}/dotnet-debian-x64.${NETCORE_1_1_RUNTIME_VERSION}.tar.gz"; \
    do \
      echo "Downloading and installing from $url" \
      && curl -SL $url --output /tmp/dotnet.tar.gz \
      && mkdir -p /usr/share/dotnet \
      && tar -zxf /tmp/dotnet.tar.gz -C /usr/share/dotnet \
      && rm /tmp/dotnet.tar.gz; \
    done

# Add Docker SDK for when building a solution that has the Docker tools project.
RUN curl -H 'Cache-Control: no-cache' -o /tmp/Microsoft.Docker.Sdk.tar.gz https://distaspnet.blob.core.windows.net/sdk/Microsoft.Docker.Sdk.tar.gz \
    && cd /usr/share/dotnet/sdk/${DOTNET_SDK_VERSION}/Sdks \
    && tar xf /tmp/Microsoft.Docker.Sdk.tar.gz \
    && rm /tmp/Microsoft.Docker.Sdk.tar.gz

# copy the ASP.NET packages manifest
COPY packagescache.csproj /tmp/warmup/

# warm up package cache
RUN dotnet restore /tmp/warmup/packagescache.csproj \
      --source https://api.nuget.org/v3/index.json \
      --verbosity quiet \
    && rm -rf /tmp/warmup/

WORKDIR /
