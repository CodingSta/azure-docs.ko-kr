---
title: 포함 파일
description: 포함 파일
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 3e02242822283ec8597a7be217228cb95e748761
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2018
---
1. 포털에서 가상 네트워크 게이트웨이를 만들려는 가상 네트워크로 이동합니다.
2. VNet 페이지의 **설정** 섹션에서 **서브넷**을 클릭하여 서브넷 페이지를 확장합니다.
3. **서브넷** 페이지의 맨 위에서 **+게이트웨이 서브넷**을 클릭하여 **서브넷 추가** 페이지를 엽니다.

  ![게이트웨이 서브넷 추가](./media/vpn-gateway-add-gwsubnet-s2s-rm-portal-include/addgwsub.png "게이트웨이 서브넷 추가")
4. 서브넷의 **이름**에 'GatewaySubnet' 값이 자동으로 채워집니다. Azure가 서브넷을 게이트웨이 서브넷으로 인식하기 위해 GatewaySubnet 값이 필요합니다. 자동으로 채워진 **주소 범위** 값을 구성 요구 사항과 일치하도록 조정합니다.

  ![게이트웨이 서브넷 추가](./media/vpn-gateway-add-gwsubnet-s2s-rm-portal-include/addsub2.png "게이트웨이 서브넷 추가")
5. 서브넷을 만들려면 페이지 아래쪽에서 **확인**을 클릭합니다.