---
title: Visual Studio Code를 사용하여 Azure IoT Edge에서 Azure Functions 디버그 | Microsoft Docs
description: VS Code에서 Azure IoT Edge를 사용하여 C# Azure Functions 디버그
services: iot-edge
keywords: ''
author: shizn
manager: timlt
ms.author: xshi
ms.date: 3/20/2018
ms.topic: article
ms.service: iot-edge
ms.openlocfilehash: 8c266a01375bf74fd4df9290255e84bc28e6089c
ms.sourcegitcommit: 5b2ac9e6d8539c11ab0891b686b8afa12441a8f3
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/06/2018
---
# <a name="use-visual-studio-code-to-debug-azure-functions-with-azure-iot-edge"></a>Visual Studio Code를 사용하여 Azure IoT Edge에서 Azure Functions 디버그

이 문서에서는 [Visual Studio Code](https://code.visualstudio.com/)를 주 개발 도구로 사용하여 IoT Edge에서 Azure Functions를 디버그하기 위한 자세한 지침을 제공합니다.

## <a name="prerequisites"></a>필수 조건
이 문서에서는 사용자가 Windows 또는 Linux를 실행하는 컴퓨터 또는 가상 머신을 개발 컴퓨터로 사용한다고 가정합니다. IoT Edge 장치가 다른 물리적 장치가 될 수도 있고, 개발 컴퓨터에서 IoT Edge 장치를 시뮬레이트할 수도 있습니다.

이 문서의 지침을 수행하기 전에 [Visual Studio Code에서 여러 모듈을 사용하여 IoT Edge 솔루션 개발](tutorial-multiple-modules-in-vscode.md)의 단계를 완료합니다. 그 후 다음 항목을 준비해야 합니다.
- 개발 컴퓨터에서 실행 중인 로컬 Docker 레지스트리. 프로토타입 및 테스트 목적에는 로컬 Docker 레지스트리를 사용하는 것이 좋습니다. 각 모듈 폴더의 `module.json` 파일에서 컨테이너 레지스트리를 업데이트할 수 있습니다.
- Azure Function 모듈 하위 폴더를 포함하는 IoT Edge 솔루션 프로젝트 작업 영역
- 함수 코드가 포함된 `run.csx` 파일
- 개발 컴퓨터에서 실행되는 Edge 런타임

## <a name="build-your-iot-edge-function-module-for-debugging-purpose"></a>디버깅을 위한 IoT Edge 함수 모듈 빌드
1. 디버깅을 시작하려면 **Dockerfile.amd64.debug**를 사용하여 docker 이미지를 다시 빌드하고 Edge 솔루션을 다시 배포해야 합니다. VS Code 탐색기에서 `deployment.template.json` 파일로 이동합니다. 끝에 `.debug`를 추가하여 함수 이미지 URL을 업데이트합니다.

    ![디버그 이미지 빌드](./media/how-to-debug-csharp-function/build-debug-image.png)

2. 솔루션을 다시 빌드합니다. VS Code 명령 팔레트에서 **Edge: Build IoT Edge solution** 명령을 입력하고 실행합니다.

3. Azure IoT Hub 장치 탐색기에서 IoT Edge 장치 ID를 마우스 오른쪽 단추로 클릭한 다음, **Edge 장치에 대한 배포 만들기**를 선택합니다. `config` 폴더 아래에서 `deployment.json`을 선택합니다. 그런 다음, VS Code 통합 터미널에서 배포 ID로 배포가 생성되었는지 확인할 수 있습니다.

> [!NOTE]
> VS Code Docker 탐색기에서 또는 터미널에서 `docker images` 명령을 실행하여 컨테이너 상태를 확인할 수 있습니다.

## <a name="start-debugging-c-function-in-vs-code"></a>VS Code에서 C# 함수 디버깅 시작
1. VS Code는 작업 영역의 `.vscode` 폴더에 위치한 `launch.json` 파일에서 구성 정보를 디버깅하도록 유지합니다. 새로운 IoT Edge 솔루션을 만들 때 이 `launch.json` 파일이 생성되었습니다. 또한 디버깅을 지원하는 새 모듈을 추가할 때마다 업데이트됩니다. 디버그 보기를 찾아서 해당 디버그 구성 파일을 선택합니다.
    ![디버그 구성 선택](./media/how-to-debug-csharp-function/select-debug-configuration.jpg)

2. `run.csx`로 이동합니다. 함수에 중단점을 추가합니다.

3. 디버깅 시작 단추를 클릭하거나 **F5** 키를 누르고, 연결할 프로세스를 선택합니다.

4. VS Code 디버그 보기의 왼쪽 패널에서 변수를 볼 수 있습니다. 


> [!NOTE]
> 위 예제에서는 컨테이너에서 .NET Core IoT Edge 함수를 디버그하는 방법을 보여 줍니다. 이 방법은 빌드하는 동안 컨테이너 이미지에 VSDBG(.NET Core 명령줄 디버거)를 포함하는 디버그 버전의 `Dockerfile.amd64.debug`를 기준으로 합니다. C# 함수의 디버그가 끝나면 프로덕션에서 사용할 준비가 완료된 IoT Edge 함수를 얻기 위해 VSDBG를 사용하지 않고 `Dockerfile`을 직접 사용하거나 사용자 지정하는 것이 좋습니다.

## <a name="next-steps"></a>다음 단계


[Visual Studio Code를 사용하여 Azure IoT Edge에서 C# 모듈 디버그](how-to-vscode-debug-csharp-module.md)

