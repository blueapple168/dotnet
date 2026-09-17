ARG REPO=ghcr.io/blueapple168/dotnet-kylinos/runtime-8.0:runtime-8.0

# Installer image
#FROM amd64/buildpack-deps:bookworm-curl AS installer
FROM $REPO AS installer
# Retrieve ASP.NET Core
RUN aspnetcore_version=8.0.31 \
    && curl --fail --show-error --location \
        --remote-name https://builds.dotnet.microsoft.com/dotnet/aspnetcore/Runtime/$aspnetcore_version/aspnetcore-runtime-$aspnetcore_version-linux-x64.tar.gz \
        --remote-name https://builds.dotnet.microsoft.com/dotnet/checksums/$aspnetcore_version-sha.txt \
    && sed -i 's/\r$//' $aspnetcore_version-sha.txt \
    && sha512sum -c $aspnetcore_version-sha.txt --ignore-missing \
    && mkdir --parents /dotnet \
    && tar --gzip --extract --no-same-owner --file aspnetcore-runtime-$aspnetcore_version-linux-x64.tar.gz --directory /dotnet ./shared/Microsoft.AspNetCore.App \
    && rm \
        aspnetcore-runtime-$aspnetcore_version-linux-x64.tar.gz \
        $aspnetcore_version-sha.txt

# .NET runtime image
FROM $REPO AS runtime

# .NET Runtime version
ENV DOTNET_VERSION=8.0.31

COPY --from=installer ["/dotnet", "/usr/share/dotnet"]

#RUN ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet
