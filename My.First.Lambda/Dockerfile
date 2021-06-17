FROM mcr.microsoft.com/dotnet/sdk:3.1 AS base
LABEL image="base"

FROM base AS dependencies
LABEL image="dependencies"
ENV MSBUILDSINGLELOADCONTEXT=1
WORKDIR /work
COPY . .
RUN dotnet add ./My.First.Lambda.Tests/My.First.Lambda.Tests.csproj package --no-restore coverlet.msbuild
RUN dotnet restore

FROM dependencies AS build
LABEL image="build"
RUN dotnet build -c Release --no-restore
RUN dotnet publish My.First.Lambda/My.First.Lambda.csproj --no-restore --no-build --configuration Release --output /app

FROM build AS unit-tests
LABEL image="unit-tests"
RUN dotnet test --no-restore \ 
                --no-build \
                --configuration Release \
                --verbosity=normal \
                --results-directory /work/TestResults \
                --logger "trx;LogFileName=test_results.xml" \
                /p:CollectCoverage=true \
                /p:CoverletOutputFormat=opencover \
                /p:CoverletOutput=/work/TestResults/opencover.xml; \
    exit 0

FROM nosinovacao/dotnet-sonar:19.12.0 AS sonarqube
WORKDIR /work
COPY --from=unit-tests /work .

FROM javieraviles/zip AS zip
WORKDIR /app
COPY --from=build /app .
