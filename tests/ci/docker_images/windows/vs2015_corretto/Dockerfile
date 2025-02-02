# escape=`

# Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
# SPDX-License-Identifier: Apache-2.0

# Keep parity with the upstream tags at https://hub.docker.com/_/microsoft-windows-servercore
FROM accp/windows_base:2019

SHELL ["cmd", "/S", "/C"]
RUN `
    # Download the Build Tools bootstrapper.
    curl -SL --output vs_buildtools.exe https://aka.ms/vs/15/release/vs_buildtools.exe `
    `
    # Install Build Tools. Build Tools for MSVC2015 are not directly downloadable without a Visual Studio
    # subscription, so we'll need to download MSVC2017 and add the MSVC2015 (VC.140) component available to
    # download with it.
    # Reference: https://docs.microsoft.com/en-us/previous-versions/visualstudio/visual-studio-2017/install/workload-component-id-vs-build-tools?view=vs-2017
    && (start /w vs_buildtools.exe --quiet --wait --norestart --nocache `
        --add Microsoft.VisualStudio.Component.VC.CLI.Support `
        --add Microsoft.VisualStudio.Component.VC.140 `
        --add Microsoft.VisualStudio.Component.VC.CMake.Project `
        || IF "%ERRORLEVEL%"=="3010" EXIT 0) `
    `
    # Cleanup
    && del /q vs_buildtools.exe

# Add msbuild and cl to PATH
RUN setx /M PATH "%PATH%;C:\Program Files (x86)\MSBuild\14.0\bin;C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin"

# Replace Windows SDK with the correct binary path. vcvarsall.bat for MSVC2015 does not account for the nested directory.
RUN rmdir /s /Q "C:\Program Files (x86)\Windows Kits\10\bin\x64" && `
    rmdir /s /Q "C:\Program Files (x86)\Windows Kits\10\bin\x86" && `
    move "C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x64" "C:\Program Files (x86)\Windows Kits\10\bin" && `
    move "C:\Program Files (x86)\Windows Kits\10\bin\10.0.17763.0\x86" "C:\Program Files (x86)\Windows Kits\10\bin"

# Download and install corretto. Installed corretto directories are automatically added to PATH.
# Corretto 8: C:\Program Files\Amazon Corretto\jdk1.8.0_332\bin
# Corretto 11: C:\Program Files\Amazon Corretto\jdk11.0.15_9\bin
# Corretto 17: C:\Program Files\Amazon Corretto\jdk17.0.3_6\bin
RUN curl -SL --output amazon-corretto-11-x64-windows-jdk.msi https://corretto.aws/downloads/resources/11.0.15.9.1/amazon-corretto-11.0.15.9.1-windows-x64.msi && `
    msiexec /i amazon-corretto-11-x64-windows-jdk.msi && `
    curl -SL --output amazon-corretto-8-windows-x64-jdk.msi https://corretto.aws/downloads/resources/8.332.08.1/amazon-corretto-8.332.08.1-windows-x64-jdk.msi && `
    msiexec /i amazon-corretto-8-windows-x64-jdk.msi && `
    curl -SL --output amazon-corretto-17-x64-windows-jdk.msi https://corretto.aws/downloads/resources/17.0.3.6.1/amazon-corretto-17.0.3.6.1-windows-x64.msi && `
    msiexec /i amazon-corretto-17-x64-windows-jdk.msi

CMD [ "cmd.exe" ]
