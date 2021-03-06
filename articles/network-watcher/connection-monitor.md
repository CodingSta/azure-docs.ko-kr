---
title: Azure Network Watcher로 네트워크 연결 모니터링 - Azure Portal | Microsoft Docs
description: Azure Portal을 사용하여 Azure Network Watcher를 통해 네트워크 연결을 모니터링하는 방법을 알아봅니다.
services: network-watcher
documentationcenter: na
author: jimdial
manager: jeconnoc
editor: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/16/2018
ms.author: jdial
ms.openlocfilehash: b0eb10d373f47191933eb1c3c22b779e9e8d6685
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 04/05/2018
---
# <a name="monitor-network-connections-with-azure-network-watcher-using-the-azure-portal"></a>Azure Portal을 사용하여 Azure Network Watcher로 네트워크 연결 모니터링

연결 모니터를 사용하여 Azure VM(가상 머신)과 IP 주소 간의 네트워크 연결을 모니터링하는 방법을 알아봅니다. 연결 모니터는 원본과 대상 IP 주소와 포트 간의 모니터링을 제공합니다. 연결 모니터를 사용하면 가상 네트워크의 VM에서 동일하거나 다른 가상 네트워크에서 포트 1433을 통해 SQL Server를 실행하는VM으로의 연결을 모니터링하는 시나리오가 가능합니다. 연결 모니터는 60초 마다 기록된 Azure Monitor 메트릭으로 연결 대기 시간을 제공합니다. 또한 홉 단위 토폴로지를 제공하고 연결에 영향을 주는 구성 문제를 식별합니다.

## <a name="prerequisites"></a>필수 조건

이 아티클의 단계를 완료하기 전에 다음 필수 구성 요소를 충족해야 합니다.

* 연결을 모니터링하려는 지역의 Network Watcher 인스턴스. 아직 없는 경우 [Azure Network Watcher 인스턴스 만들기](network-watcher-create.md)의 단계를 완료하여 만듭니다.
* 모니터링하는 VM입니다. VM을 만드는 방법을 알아보려면 [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) 또는 [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) VM 만들기를 참조하세요.
* 연결을 모니터링할 VM에 `AzureNetworkWatcherExtension`을 설치합니다. Windows VM에서 확장을 설치하려면 [Windows용 Azure Network Watcher 에이전트 가상 머신 확장](../virtual-machines/windows/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)을 참조하고, Linux VM에서 확장을 설치하려면 [Linux용 Azure Network Watcher 에이전트 가상 머신 확장](../virtual-machines/linux/extensions-nwa.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json)을 참조하세요. 확장은 모니터링하려는 대상 엔드포인트에서 필요하지 않습니다.

## <a name="sign-in-to-azure"></a>Azure에 로그인 

[Azure 포털](http://portal.azure.com)에 로그인합니다.

## <a name="create-a-connection-monitor"></a>연결 모니터 만들기

다음 단계를 사용하여 포트 80 및 1433을 통한 대상 VM에 대한 연결 모니터링을 수행할 수 있습니다.

1. 포털의 왼쪽에서 **모든 서비스**를 선택합니다.
2. **필터** 상자에 *Network Watcher*를 입력합니다. 검색 결과에 **Network Watcher**가 나타나면 이를 선택합니다.
3. **모니터링** 아래에서 **연결 모니터**를 선택합니다.
4. **+추가**를 선택합니다.
5. 모니터링하려는 연결에 대한 정보를 입력하거나 선택한 다음, **추가**를 선택합니다. 다음 그림에 나와 있는 예에서는 포트 80을 통해 *MultiTierApp0* VM에서 *Database0* VM으로 연결을 모니터링합니다.

    ![연결 모니터 추가](./media/connection-monitor/add-connection-monitor.png)

    모니터링을 시작합니다. 연결 모니터는 60초마다 프로브합니다.
6. 동일한 원본 및 대상 VM 및 다음 값을 지정하여 5단계를 다시 완료합니다.
    
    |설정  |값          |
    |---------|---------      |
    |Name     | AppToDB(1433) |
    |포트     | 1433          |

## <a name="view-connection-monitoring"></a>연결 모니터링 보기

1. 연결 모니터링을 보려면 [연결 모니터 만들기](#create-a-connection-monitor)의 1-3단계를 완료합니다.
2. 다음 그림에는 *AppToDB(80)* 연결에 대한 세부 정보가 나와 있습니다. **상태**는 연결할 수 있습니다. **그래프 보기**는 **평균 왕복 시간** 및 **% 프로브 실패**를 나타냅니다. 그래프는 홉 단위 정보를 제공하고 대상 연결 가능성에 영향을 주는 문제가 없음을 표시합니다.

    ![그래프 보기](./media/connection-monitor/view-graph.png)

3. 다음 그림에 나온 *AppToDB(1433)* 연결을 보면 동일한 원본 및 대상 VM에 대해 상태가 포트 1433을 통해 접근할 수 없음을 확인할 수 있습니다. 이 시나리오의 **그리드 보기**는 홉 단위 정보와 연결 가능성에 영향을 미치는 문제를 제공합니다. 이 경우 NSG 규칙이 두 번째 홉에 있는 포트 1433에서 모든 트래픽을 차단합니다.

    ![그리드 보기](./media/connection-monitor/view-grid.png)

## <a name="next-steps"></a>다음 단계

- [경고로 트리거된 패킷 캡처를 만들어](network-watcher-alert-triggered-packet-capture.md) VM 경고로 패킷 캡처를 자동화하는 방법을 알아봅니다.
- [IP 흐름 확인 검사](network-watcher-check-ip-flow-verify-portal.md)를 방문하여 VM에서 또는 가상 머신으로 특정 트래픽 전송이 허용되는지 확인합니다.