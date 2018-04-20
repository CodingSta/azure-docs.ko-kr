---
title: Azure Storage에서 SAS(공유 액세스 서명) 사용 | Microsoft 문서
description: SAS(공유 액세스 서명)를 사용하여 Blob, 큐, 테이블 및 파일을 비롯한 Azure Storage 리소스에 대한 액세스 권한을 위임하는 방법을 알아봅니다.
services: storage
documentationcenter: ''
author: craigshoemaker
manager: jeconnoc
editor: tysonn
ms.assetid: 46fd99d7-36b3-4283-81e3-f214b29f1152
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 04/18/2017
ms.author: cshoe
ms.openlocfilehash: d3f8b3261f9e2e86dbcaa41b92111545abeffe54
ms.sourcegitcommit: d74657d1926467210454f58970c45b2fd3ca088d
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/28/2018
---
# <a name="using-shared-access-signatures-sas"></a>SAS(공유 액세스 서명) 사용

SAS(공유 액세스 서명)를 사용하면 계정 키를 노출하지 않고 다른 클라이언트에게 저장소 계정의 개체에 대한 제한된 액세스 권한을 부여할 수 있습니다. 이 문서에서는 SAS 모델의 개요를 설명하고 SAS 모범 사례를 검토하고 몇 가지 예를 살펴봅니다.

여기에 제시된 것 이외의 SAS를 사용하는 추가 코드 예제는 [.NET에서 Azure Blob Storage 시작](https://azure.microsoft.com/documentation/samples/storage-blob-dotnet-getting-started/) 및 [Azure 코드 샘플](https://azure.microsoft.com/documentation/samples/?service=storage) 라이브러리에서 사용할 수 있는 다른 샘플을 참조하세요. GitHub에서 샘플 응용 프로그램을 다운로드하고 실행하거나 코드를 탐색할 수 있습니다.

## <a name="what-is-a-shared-access-signature"></a>공유 액세스 서명이란?
공유 액세스 서명은 저장소 계정의 리소스에 대한 위임된 권한을 제공합니다. SAS로 계정 키를 공유하지 않고 저장 계정의 리소스에 대한 클라이언트의 액세스를 승인할 수 있습니다. 이는 응용 프로그램에서 공유 액세스 서명을 사용하는 중요한 점입니다. SAS는 선택키를 손상시키지 않고 저장 리소스를 공유할 수 있는 보안 방법입니다.

[!INCLUDE [storage-account-key-note-include](../../../includes/storage-account-key-note-include.md)]

SAS가 있는 클라이언트에게 허용하는 액세스 유형에 대해 SAS는 세부적인 제어를 제공합니다.

* SAS가 유효한 경우 시작 시간 및 만료 시간을 포함하는 간격입니다.
* SAS에서 부여된 권한입니다. 예를 들어 blob의 SAS는 해당 blob에 읽기 및 쓰기 권한을 부여할 수 있지만 삭제 권한은 부여하지 않습니다.
* Azure Storage가 SAS를 수락하는 선택적 IP 주소 또는 IP 주소 범위입니다. 예를 들어 조직에 속한 IP 주소 범위를 지정할 수 있습니다.
* Azure Storage가 SAS을 수락하는 프로토콜입니다. HTTPS를 사용하여 클라이언트에 대한 액세스를 제한하려면 이 선택적 매개 변수를 사용할 수 있습니다.

## <a name="when-should-you-use-a-shared-access-signature"></a>공유 액세스 서명은 언제 사용하나요?
저장소 계정의 액세스 키가 없는 클라이언트에게 저장소 계정의 리소스에 대한 액세스 권한을 제공하려는 경우에 SAS를 사용할 수 있습니다. 저장소 계정 키는 기본 키와 보조 액세스 키를 모두 포함하여, 계정과 계정에 있는 모든 리소스에 대한 관리 권한을 부여합니다. 이러한 키를 노출하면 계정도 악의적이거나 잘못된 용도로 사용될 수 있습니다. 공유 액세스 서명은 클라이언트가 계정 키를 사용하지 않고 명시적으로 부여된 권한에 따라 저장소 계정에서 데이터를 읽고, 쓰고, 삭제하도록 허용하는 안전한 대체 방법입니다.

SAS가 유용한 일반적인 시나리오로는 다른 사용자가 저장소 계정에서 데이터를 읽고 쓰는 서비스가 있습니다. 저장소 계정에 사용자 데이터를 저장하는 시나리오에는 다음과 같은 두 가지 일반적인 디자인 패턴이 있습니다.

1. 클라이언트는 인증을 수행하는 프런트 엔드 프록시 서비스를 통해 데이터를 업로드하고 다운로드합니다. 프런트 엔드 프록시 서비스는 비즈니스 규칙의 유효성을 검사할 수 있다는 장점이 있지만, 데이터의 양이 많거나 트랜잭션 볼륨이 높은 경우 수요에 맞게 확장 가능한 서비스를 구축하려면 많은 비용이 필요하거나 어려움이 따를 수 있습니다.

  ![시나리오 다이어그램: 프런트 엔드 프록시 서비스](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-fe-proxy-service.png)   

1. 간단한 서비스에서 필요에 따라 클라이언트를 인증한 다음 SAS를 생성합니다. 클라이언트는 SAS를 수신한 후 SAS에 정의된 권한을 사용하여 SAS에 허용된 간격으로 저장소 계정 리소스에 직접 액세스할 수 있습니다. SAS를 사용하면 프런트 엔드 프록시 서비스를 통해 모든 데이터를 라우팅할 필요성이 감소됩니다.

  ![시나리오 다이어그램: SAS 공급자 서비스](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-provider-service.png)   

대부분의 실제 서비스에서는 이러한 두 가지 방법을 혼합하여 사용할 수 있습니다. 예를 들어 일부 데이터는 프런트 엔드 프록시를 통해 처리 및 확인하고 일부 데이터는 SAS를 사용하여 직접 저장하거나 읽습니다.

또한 특정 시나리오에서는 SAS를 사용하여 복사 작업의 원본 개체를 인증해야 합니다.

* 다른 저장소 계정에 있는 다른 Blob에 Blob을 복사하는 경우 SAS를 사용하여 원본 Blob을 인증해야 합니다. 필요에 따라 SAS를 사용하여 대상 Blob도 인증할 수 있습니다.
* 다른 저장소 계정에 있는 다른 파일에 파일을 복사하는 경우 SAS를 사용하여 원본 파일을 인증해야 합니다. 필요에 따라 SAS를 사용하여 대상 파일도 인증할 수 있습니다.
* Blob을 파일에 복사하거나 파일을 Blob에 복사하는 경우 원본 및 대상 개체가 동일한 저장소 계정 내에 있더라도 SAS를 사용하여 원본 개체를 인증해야 합니다.

## <a name="types-of-shared-access-signatures"></a>공유 액세스 서명의 유형
두 가지 유형의 공유 액세스 서명을 만들 수 있습니다.

* **서비스 SAS** 서비스 SAS는 저장소 서비스(Blob, 큐, 테이블, 파일 서비스) 중 하나의 리소스에만 액세스 권한을 위임합니다. 서비스 SAS 토큰을 구성하는 방법에 대한 자세한 내용은 [서비스 SAS 구성](https://msdn.microsoft.com/library/dn140255.aspx) 및 [서비스 SAS 예제](https://msdn.microsoft.com/library/dn140256.aspx)를 참조하세요.
* **계정 SAS** 계정 SAS는 하나 이상의 저장소 서비스에서 리소스에 대한 액세스 권한을 위임합니다. 서비스 SAS를 통해 사용 가능한 모든 작업은 계정 SAS를 통해서도 사용할 수 있습니다. 또한 계정 SAS를 사용하면 **서비스 속성 가져오기/설정**, **서비스 통계 가져오기**와 같은 특정 서비스에 적용되는 작업에 대한 액세스 권한을 위임할 수 있습니다. 또한 서비스 SAS로 허용되지 않는 Blob 컨테이너, 테이블, 큐, 파일 공유에서 읽기, 쓰기, 삭제 작업에 대한 액세스 권한을 위임할 수 있습니다. 계정 SAS 토큰을 구성하는 방법에 대한 자세한 내용은 [계정 SAS 구성](https://msdn.microsoft.com/library/mt584140.aspx) 을 참조하세요.

## <a name="how-a-shared-access-signature-works"></a>공유 액세스 서명 사용 방법
공유 액세스 서명은 하나 이상의 저장소 리소스를 가리키는 서명된 URI이며 쿼리 매개 변수의 특별 집합이 포함된 토큰이 들어 있습니다. 토큰은 클라이언트가 리소스를 액세스하는 방식을 나타냅니다. 이러한 쿼리 매개 변수 중 하나인 서명은 SAS 매개 변수에서 구성되고 계정 키로 서명됩니다. 이 서명은 Azure Storage에서 SAS를 인증하는 데 사용됩니다.

다음은 리소스 URI과 SAS 토큰을 표시하는 SAS URI의 예입니다.

![SAS URI의 구성 요소](./media/storage-dotnet-shared-access-signature-part-1/sas-storage-uri.png)   

SAS 토큰은 *클라이언트* 쪽에서 생성된 문자열입니다. 코드 예제는 [SAS 예제](#sas-examples) 섹션을 참조하세요. 저장소 클라이언트 라이브러리를 사용하여 생성하는 SAS 토큰은 어떤 방식으로든 Azure Storage에 의해 추적되지 않습니다. 클라이언트 쪽에서 SAS 토큰 개수를 제한 없이 만들 수 있습니다.

클라이언트가 Azure Storage에 SAS URI를 요청의 일부로 제공하는 경우 서비스는 SAS 매개 변수와 서명을 확인하여 요청을 인증하기에 유효한지 확인합니다. 서비스에서 서명이 유효하다고 확인한 경우 요청이 인증됩니다. 그렇지 않은 경우 오류 코드 403(금지됨) 요청이 거부됩니다.

## <a name="shared-access-signature-parameters"></a>공유 액세스 서명 매개 변수
계정 SAS 및 서비스 SAS 토큰에는 일반적인 매개 변수 몇 개가 포함되며 다른 몇 가지 매개 변수도 사용합니다.

### <a name="parameters-common-to-account-sas-and-service-sas-tokens"></a>계정 SAS 및 서비스 SAS 토큰에 일반적인 매개 변수
* **Api 버전** 요청을 실행하기 위해 사용할 저장소 서비스 버전을 지정하는 선택적 매개 변수입니다.
* **서비스 버전** 요청을 인증하기 위해 사용할 저장소 서비스 버전을 지정하는 선택적 매개 변수입니다.
* **시작 시간.** SAS가 유효해지는 시간입니다. 공유 액세스 서명의 시작 시간은 선택 사항입니다. 시작 시간이 생략된 경우 SAS가 즉시 유효합니다. 시작 시간은 특정 UTC 지정자("Z")를 이용하여 UTC(협정 세계시)로 표시해야 합니다(예: `1994-11-05T13:15:30Z`).
* **만료 시간.** SAS가 더 이상 유효하지 않게 되는 시간입니다. 모범 사례에 따라 SAS의 만료 시간을 지정하거나 만료 시간을 저장된 액세스 정책과 연결하는 것이 좋습니다. 만료 시간은 특정 UTC 지정자("Z")를 이용하여 UTC(협정 세계시)로 표시해야 합니다(예: `1994-11-05T13:15:30Z`). 자세한 내용은 아래를 참조하세요.
* **사용 권한** SAS에 지정된 사용 권한은 클라이언트가 SAS를 사용하여 저장소 리소스에 대해 수행할 수 있는 작업을 나타냅니다. 사용 가능한 권한은 계정 SAS와 서비스 SAS가 다릅니다.
* **IP** 요청을 수락할 Azure 외부(Express 경로에 대한 [라우팅 세션 구성 상태](../../expressroute/expressroute-workflows.md#routing-session-configuration-state) 섹션 참조)의 IP 주소 또는 IP 주소 범위를 지정하는 선택적 매개 변수입니다.
* **프로토콜** 요청에 허용되는 프로토콜을 지정하는 선택적 매개 변수입니다. 기본값인 HTTPS 및 HTTP(`https,http`) 또는 HTTPS만(`https`) 허용됩니다. HTTP만은 허용되는 값이 아닙니다.
* **서명** 서명은 토큰의 일부로 지정된 다음 암호화된 다른 매개 변수에서 구성됩니다. 서명은 SAS 인증에 사용됩니다.

### <a name="parameters-for-a-service-sas-token"></a>서비스 SAS 토큰의 매개 변수
* **저장소 리소스** 서비스 SAS를 사용하여 액세스 권한을 위임할 수 있는 저장소 리소스는 다음과 같습니다.
  * 컨테이너 및 Blob
  * 파일 공유 및 파일
  * 큐
  * 테이블 및 테이블 엔터티 범위

### <a name="parameters-for-an-account-sas-token"></a>계정 SAS 토큰의 매개 변수
* **서비스** 계정 SAS는 하나 이상의 저장소 서비스에 대한 액세스 권한을 위임할 수 있습니다. 예를 들어 Blob 및 파일 서비스에 대한 액세스 권한을 위임하는 계정 SAS를 만들 수 있습니다. 또는 4개 서비스(Blob, 큐, 테이블, 파일) 전체에 대한 액세스 권한을 위임하는 SAS를 만들 수 있습니다.
* **저장소 리소스 유형** 계정 SAS는 특정 리소스가 아닌 하나 이상의 클래스의 저장소 리소스에 적용됩니다. 계정 SAS를 만들어 다음에 대한 액세스 권한을 위임할 수 있습니다.
  * 저장소 계정 리소스에 대해 호출되는 서비스 수준 API. 예를 들면 **서비스 속성 가져오기/설정**, **서비스 통계 가져오기**, **컨테이너/큐/테이블/공유 나열**이 포함됩니다.
  * 각 서비스(Blob 컨테이너, 큐, 테이블, 파일 공유)의 컨테이너 개체에 대해 호출되는 컨테이너 수준 API입니다. 예를 들어 **컨테이너 만들기/삭제**, **큐 만들기/삭제**, **테이블 만들기/삭제**, **공유 만들기/삭제**, **Blob/파일 및 디렉터리 나열**이 있습니다.
  * Blob, 큐 메시지, 테이블 엔터티, 파일에 대해 호출되는 개체 수준 API. 예를 들어 **Blob 배치**, **쿼리 엔터티**, **메시지 가져오기**, **파일 만들기**가 있습니다.

## <a name="examples-of-sas-uris"></a>SAS URI의 예

### <a name="service-sas-uri-example"></a>서비스 SAS URI 예

다음은 Blob에 대한 읽기 및 쓰기 권한을 제공하는 서비스 SAS URI의 예입니다. 테이블에서는 URI의 각 부분이 SAS에서 어떤 역할을 하는지를 이해할 수 있도록 세분화했습니다.

```
https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt?sv=2015-04-05&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D
```

| Name | SAS 부분 | 설명 |
| --- | --- | --- |
| Blob URI |`https://myaccount.blob.core.windows.net/sascontainer/sasblob.txt` |Blob의 주소입니다. HTTPS를 사용하는 것이 좋습니다. |
| Storage 서비스 버전 |`sv=2015-04-05` |2012-02-12 이후의 저장소 서비스 버전의 경우 이 매개 변수는 사용할 버전을 나타냅니다. |
| 시작 시간 |`st=2015-04-29T22%3A18%3A26Z` |UTC 시간으로 지정됩니다. SAS를 즉시 유효화하려면 시작 시간을 생략하십시오. |
| 만료 시간 |`se=2015-04-30T02%3A23%3A26Z` |UTC 시간으로 지정됩니다. |
| 리소스 |`sr=b` |Blob의 리소스입니다. |
| 권한 |`sp=rw` |SAS에서 부여하는 권한에는 읽기 및 쓰기가 포함됩니다. |
| IP 범위 |`sip=168.1.5.60-168.1.5.70` |요청을 수락할 IP 주소 범위입니다. |
| 프로토콜 |`spr=https` |HTTPS를 사용하는 요청만 허용됩니다. |
| 서명 |`sig=Z%2FRHIX5Xcg0Mq2rqI3OlWTjEg2tYkboXr1P9ZUXDtkk%3D` |Blob에 대한 액세스 권한을 인증하는 데 사용합니다. 이 서명은 SHA256 알고리즘을 사용하여 서명할 문자열과 키를 통해 계산되고 Base64 인코딩을 사용하여 인코드되는 HMAC입니다. |

### <a name="account-sas-uri-example"></a>계정 SAS URI 예

다음은 토큰에 동일한 일반적 매개 변수를 사용하는 계정 SAS의 예입니다. 이러한 매개 변수는 위에서 설명했으므로 여기에서는 설명을 생략합니다. 계정 SAS에 해당하는 매개 변수만 아래 표에 설명되어 있습니다.

```
https://myaccount.blob.core.windows.net/?restype=service&comp=properties&sv=2015-04-05&ss=bf&srt=s&st=2015-04-29T22%3A18%3A26Z&se=2015-04-30T02%3A23%3A26Z&sr=b&sp=rw&sip=168.1.5.60-168.1.5.70&spr=https&sig=F%6GRVAZ5Cdj2Pw4tgU7IlSTkWgn7bUkkAg8P6HESXwmf%4B
```

| Name | SAS 부분 | 설명 |
| --- | --- | --- |
| 리소스 URI |`https://myaccount.blob.core.windows.net/?restype=service&comp=properties` |서비스 속성을 가져오거나(GET으로 호출할 경우) 서비스 속성을 설정하기 위한(SET으로 호출하는 경우) 매개 변수를 사용하는 Blob service 끝점입니다. |
| Services |`ss=bf` |SAS는 Blob 및 파일 서비스에 적용됩니다. |
| 리소스 유형 |`srt=s` |SAS는 서비스 수준 작업에 적용됩니다. |
| 권한 |`sp=rw` |사용 권한으로 읽기 및 쓰기 작업에 대한 액세스 권한을 부여합니다. |

사용 권한이 서비스 수준으로 제한된 경우 이 SAS로 액세스 가능한 작업은 **Blob service 속성 가져오기**(읽기) 및 **Blob service 속성 설정**(쓰기)입니다. 하지만 다른 리소스 URI를 사용하면 동일한 SAS 토큰을 사용하여 **Blob 서비스 통계 가져오기** (읽기)에 대한 액세스 권한을 위임할 수도 있습니다.

## <a name="controlling-a-sas-with-a-stored-access-policy"></a>저장된 액세스 정책을 사용하여 SAS 관리
공유 액세스 서명은 다음 두 가지 형식 중 하나를 사용할 수 있습니다.

* **임시 SAS:** 임시 SAS를 만들 때 SAS의 시작 시간, 만료 시간 및 사용 권한이 SAS URI에 모두 지정되거나 시작 시간이 생략되는 경우에는 암시적으로 지정됩니다. 이 SAS 유형은 계정 SAS 또는 서비스 SAS로 만들 수 있습니다.
* **저장된 액세스 정책 사용 SAS:** 저장된 액세스 정책은 리소스 컨테이너(Blob 컨테이너, 테이블, 큐 또는 파일 공유)에서 정의되며, 하나 이상의 공유 액세스 서명에 대한 제약 조건을 관리하는 데 사용할 수 있습니다. SAS를 공유 액세스 정책과 연결할 경우 SAS는 저장된 액세스 정책에 대해 정의된 제약 조건(시작 시간, 만료 시간 및 사용 권한)을 상속합니다.

> [!NOTE]
> 현재, 계정 SAS는 애드혹 SAS여야 합니다. 저장된 액세스 정책은 아직 계정 SAS에 지원되지 않습니다.

두 형식의 차이점은 주요 시나리오인 해지에 중요합니다. SAS URI는 URL이므로 원래 작성자에 관계없이 SAS를 가져오는 모든 사용자가 이를 사용할 수 있습니다. SAS가 공개적으로 게시된 경우 전 세계의 모든 사용자가 SAS를 사용할 수 있습니다. SAS는 다음 네 가지 중 하나에 해당할 때까지 소유하고 있는 모든 사용자에게 리소스에 대한 액세스 권한을 부여합니다.

1. SAS에 지정된 만료 시간에 도달한 경우
2. SAS에서 참조되는 저장된 액세스 정책에 지정된 만료 시간에 도달한 경우(저장된 액세스 정책을 참조하고 이 정책에 만료 시간이 지정된 경우). 시간 간격이 경과하거나 저장된 액세스 정책에서 만료 시간을 과거의 시간으로 수정한 경우에 발생할 수 있으며 이는 SAS를 해지하는 한 가지 방법입니다.
3. SAS에서 참조되는 저장된 액세스 정책을 삭제한 경우(SAS를 해지하는 다른 방법). 동일한 이름을 사용하여 저장된 액세스 정책을 다시 만들면 저장된 액세스 정책에 연결된 사용 권한에 따라 모든 기존 SAS 토큰이 다시 유효해집니다(SAS의 만료 시간이 경과하지 않은 것으로 가정). SAS를 해지하기 위해 만료 시간을 미래의 시간으로 지정하여 액세스 정책을 다시 만들 경우 다른 이름을 사용해야 합니다.
4. SAS를 만드는 데 사용된 계정 키를 다시 생성한 경우. 계정 키를 다시 생성하면 해당 키를 사용하는 모든 응용 프로그램 구성 요소는 다른 유효한 계정 키 또는 새로 생성된 계정 키를 사용하도록 업데이트될 때까지 인증되지 않습니다.

> [!IMPORTANT]
> 공유 액세스 서명 URI는 서명을 만드는 데 사용된 계정 키 및 저장된 관련 액세스 정책(있는 경우)에 연결됩니다. 저장된 액세스 정책을 지정하지 않는 경우 공유 액세스 서명을 해지하는 방법은 계정 키를 변경하는 것뿐입니다.

## <a name="authenticating-from-a-client-application-with-a-sas"></a>SAS를 사용하여 클라이언트 응용 프로그램 인증
SAS를 소유하는 클라이언트는 SAS를 사용하여 계정 키를 소유하지 않는 저장소 계정에 대해 요청을 인증할 수 있습니다. SAS는 연결 문자열에 포함되거나 적절한 생성자 또는 메서드에서 직접 사용될 수 있습니다.

### <a name="using-a-sas-in-a-connection-string"></a>연결 문자열에서 SAS 사용
[!INCLUDE [storage-use-sas-in-connection-string-include](../../../includes/storage-use-sas-in-connection-string-include.md)]

### <a name="using-a-sas-in-a-constructor-or-method"></a>생성자 또는 메서드에서 SAS 사용
여러 Azure Storage 클라이언트 라이브러리 생성자와 메서드 오버로드는 SAS 매개 변수를 제공하므로 SAS를 사용하는 서비스에 대한 요청을 인증할 수 있습니다.

예를 들어, 여기서 블록 blob에 대한 참조를 만드는 데 SAS URI가 사용됩니다. SAS는 요청에 필요한 유일한 자격 증명을 제공합니다. 그런 다음 블록 blob 참조는 쓰기 작업에 사용됩니다.

```csharp
string sasUri = "https://storagesample.blob.core.windows.net/sample-container/" +
    "sampleBlob.txt?sv=2015-07-08&sr=b&sig=39Up9JzHkxhUIhFEjEH9594DJxe7w6cIRCg0V6lCGSo%3D" +
    "&se=2016-10-18T21%3A51%3A37Z&sp=rcw";

CloudBlockBlob blob = new CloudBlockBlob(new Uri(sasUri));

// Create operation: Upload a blob with the specified name to the container.
// If the blob does not exist, it will be created. If it does exist, it will be overwritten.
try
{
    MemoryStream msWrite = new MemoryStream(Encoding.UTF8.GetBytes(blobContent));
    msWrite.Position = 0;
    using (msWrite)
    {
        await blob.UploadFromStreamAsync(msWrite);
    }

    Console.WriteLine("Create operation succeeded for SAS {0}", sasUri);
    Console.WriteLine();
}
catch (StorageException e)
{
    if (e.RequestInformation.HttpStatusCode == 403)
    {
        Console.WriteLine("Create operation failed for SAS {0}", sasUri);
        Console.WriteLine("Additional error information: " + e.Message);
        Console.WriteLine();
    }
    else
    {
        Console.WriteLine(e.Message);
        Console.ReadLine();
        throw;
    }
}

```

## <a name="best-practices-when-using-sas"></a>SAS를 사용하는 경우 모범 사례
응용 프로그램에서 공유 액세스 서명을 사용할 경우 다음과 같은 두 가지 잠재적 위험에 대해 잘 알고 있어야 합니다.

* SAS가 누설될 경우 SAS를 획득한 모든 사용자가 SAS를 사용하여 저장소 계정을 손상시킬 수 있습니다.
* 클라이언트 응용 프로그램에 제공된 SAS가 만료되어 응용 프로그램이 서비스에서 새 SAS를 검색할 수 없는 경우 응용 프로그램의 기능이 저하될 수 있습니다.

다음은 공유 액세스 서명을 사용하여 이러한 위험을 완화하는 데 도움이 될 수 있는 권장 사항입니다.

1. **항상 HTTPS를 사용하여** SAS를 만들거나 배포합니다. HTTP를 통해 SAS를 전달할 경우 SAS가 노출되면 메시지 가로채기(man-in-the-middle) 공격을 수행하는 공격자가 SAS를 읽은 다음 의도된 사용자처럼 SAS를 사용할 수 있으므로, 악의적인 사용자가 중요한 데이터를 손상시킬 수 있습니다.
2. **저장된 액세스 정책을 최대한 참조합니다.** 저장된 액세스 정책을 사용하면 저장소 계정 키를 다시 생성할 필요 없이 사용 권한을 해지할 수 있습니다. 만료 시간을 매우 길게(또는 무제한) 설정하고 정기적으로 업데이트하여 만료 시간을 미래의 시간으로 변경시켜야 합니다.
3. **임시 SAS의 경우 짧은 만료 시간을 사용합니다.** 이렇게하면 SAS가 손상되더라도 단기적으로만 유효합니다. 이 방법은 저장된 액세스 정책을 참조할 수 없는 경우에 특히 중요합니다. 또한 짧은 만료 시간은 업로드할 수 있는 시간을 제한하여 Blob에 쓸 수 있는 데이터의 양을 제한할 수 있습니다.
4. **필요한 경우 클라이언트가 SAS를 자동으로 갱신하도록 허용합니다.** 클라이언트는 만료일 이전에 SAS를 갱신하여 SAS를 제공하는 서비스를 사용할 수 없을 때 다시 시도할 수 있는 시간적 여유를 확보해야 합니다. 만료 기간 내에 완료할 수 있는 즉각적인 소규모 단기 작업에 SAS를 사용할 경우에는 SAS를 갱신할 필요가 없습니다. 하지만 클라이언트가 SAS를 통해 일상 요청을 수행하는 경우에는 중간에 만료될 수 있습니다. 따라서 이전에 언급한 SAS 단기 실행 요구 사항과 클라이언트가 조기에 갱신을 요청하여 갱신 이전에 SAS가 만료되어 가동 중단이 발생하는 것을 방지해야 하는 요구 사항을 적절하게 조율하는 것이 중요합니다.
5. **SAS 시작 시간에 유의하세요.** SAS 시작 시간을 **지금**으로 설정한 경우 클럭 오차(컴퓨터의 차이에 따른 현재 시간의 차이)로 인해 처음 몇 분 동안 간헐적으로 오류가 발생할 수 있습니다. 일반적으로 시작 시간을 최소 15분 이전으로 설정하거나 설정하지 않습니다. 그러면 시작 시간이 즉시 유효해집니다. 이는 만료 시간에도 일반적으로 적용됩니다. 요청 방향에서 최대 15분의 클럭 오차가 발생할 수 있습니다. 2012-02-12 이전 REST 버전을 사용하는 클라이언트의 경우 저장된 액세스 정책을 참조하지 않는 SAS의 최대 기간은 1시간이고, 이 시간보다 더 긴 기간을 지정하는 정책은 실패합니다.
6. **액세스할 리소스를 구체적으로 지정하세요.** 보안을 위한 최적의 방법은 사용자에게 필요한 최소 권한만을 제공하는 것입니다. 사용자가 단일 엔터티에 대한 읽기 권한만 필요한 경우 해당 엔터티에 대한 읽기 권한만 부여하고 모든 엔터티에 대한 읽기/쓰기/삭제 권한은 부여하지 않습니다. 또한 SAS가 공격자의 수중에 넘어갈 가능성이 낮아지므로 SAS가 손상될 경우 손해가 줄어듭니다.
7. **SAS 사용 작업을 포함한 모든 사용량에 대한 요금이 계정에 청구됩니다.** Blob에 쓰기 권한을 제공한 경우 사용자가 200GB Blob을 업로드하도록 선택할 수 있습니다. 사용자에게 읽기 권한도 제공한 경우 사용자가 이 Blob을 10번 다운로드하도록 선택하여 2TB의 발신 비용이 청구될 수 있습니다. 또한 제한된 권한을 제공하여 악의적인 사용자의 작업 가능성을 낮추세요. 단기 실행 SAS를 사용하여 이 위협을 줄이세요. 이때 종료 시간의 클럭 오차에 유의하세요.
8. **SAS를 사용하여 작성된 데이터의 유효성을 검사하세요.** 클라이언트 응용 프로그램이 저장소 계정에 데이터를 쓸 경우 해당 데이터에 문제가 있을 수 있습니다. 데이터를 사용할 준비가 되기 이전에 응용 프로그램에서 데이터의 유효성을 검사하거나 권한을 부여해야 하는 경우 데이터를 작성한 이후 응용 프로그램에서 데이터를 사용하기 이전에 유효성 검사를 수행해야 합니다. 그러면 SAS를 적절한 방법으로 습득한 사용자나 누설된 SAS를 악용하는 사용자로 인해 계정이 손상되거나 데이터에 악의적인 데이터가 기록되는 것을 방지할 수 있습니다.
9. **경우에 따라 SAS를 사용하지 마세요..** 저장소 계정에 대한 특정 작업 관련 위험이 SAS의 이점을 능가하는 경우도 있습니다. 그런 작업에 대해서는 비즈니스 규칙 유효성 검사, 인증 및 감사를 수행한 이후에 저장소 계정에 쓰는 중간 계층 서비스를 만듭니다. 또한 다른 방법으로 액세스를 관리하는 것이 더 간단한 경우도 있습니다. 예를 들어 컨테이너의 모든 Blob을 공개적으로 읽기 가능하도록 설정하려면 모든 클라이언트가 액세스하도록 SAS를 제공하는 대신에 컨테이너를 공용으로 설정할 수 있습니다.
10. **저장소 분석을 사용하여 응용 프로그램을 모니터링합니다.** 로깅 및 메트릭을 사용하여 SAS 공급자 서비스의 가동 중단이나 저장된 액세스 정책의 잘못된 제거로 인한 인증 오류의 급격한 증가를 관찰할 수 있습니다. 자세한 내용은 [Azure Storage 팀 블로그](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/08/03/windows-azure-storage-logging-using-logs-to-track-storage-requests.aspx) (영문)를 참조하십시오.

## <a name="sas-examples"></a>SAS 예제
다음은 공유 액세스 서명의 두 유형(계정 SAS, 서비스 SAS)에 대한 몇 가지 예입니다.

이러한 C# 예제를 실행하려면 프로젝트에서 다음 NuGet 패키지를 참조해야 합니다.

* [Azure Storage Client Library for .NET](http://www.nuget.org/packages/WindowsAzure.Storage), 버전 6.x 이상(계정 SAS 사용을 위해).
* [Azure 구성 관리자](http://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager)

SAS를 만들고 테스트하는 방법을 보여 주는 추가 예제는 [저장소에 대한 Azure 코드 샘플](https://azure.microsoft.com/documentation/samples/?service=storage)을 참조하세요.

### <a name="example-create-and-use-an-account-sas"></a>예제: SAS 계정 만들기 및 사용
다음 코드 예제는 Blob 및 파일 공유에 유효한 계정 SAS를 만들며 클라이언트가 읽기, 쓰기, 목록 권한을 사용하여 서비스 수준 API에 액세스할 수 있는 권한을 부여합니다. 계정 SAS는 프로토콜을 HTTPS로 제한하므로 반드시 HTTPS로 요청해야 합니다.

```csharp
static string GetAccountSASToken()
{
    // To create the account SAS, you need to use your shared key credentials. Modify for your account.
    const string ConnectionString = "DefaultEndpointsProtocol=https;AccountName=account-name;AccountKey=account-key";
    CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConnectionString);

    // Create a new access policy for the account.
    SharedAccessAccountPolicy policy = new SharedAccessAccountPolicy()
        {
            Permissions = SharedAccessAccountPermissions.Read | SharedAccessAccountPermissions.Write | SharedAccessAccountPermissions.List,
            Services = SharedAccessAccountServices.Blob | SharedAccessAccountServices.File,
            ResourceTypes = SharedAccessAccountResourceTypes.Service,
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Protocols = SharedAccessProtocol.HttpsOnly
        };

    // Return the SAS token.
    return storageAccount.GetSharedAccessSignature(policy);
}
```

계정 SAS를 사용하여 Blob service에 대한 서비스 수준 API에 액세스하려면 SAS를 사용하는 Blob 클라이언트 개체와 저장소 계정의 Blob 저장소 끝점을 구성합니다.

```csharp
static void UseAccountSAS(string sasToken)
{
    // Create new storage credentials using the SAS token.
    StorageCredentials accountSAS = new StorageCredentials(sasToken);
    // Use these credentials and the account name to create a Blob service client.
    CloudStorageAccount accountWithSAS = new CloudStorageAccount(accountSAS, "account-name", endpointSuffix: null, useHttps: true);
    CloudBlobClient blobClientWithSAS = accountWithSAS.CreateCloudBlobClient();

    // Now set the service properties for the Blob client created with the SAS.
    blobClientWithSAS.SetServiceProperties(new ServiceProperties()
    {
        HourMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        MinuteMetrics = new MetricsProperties()
        {
            MetricsLevel = MetricsLevel.ServiceAndApi,
            RetentionDays = 7,
            Version = "1.0"
        },
        Logging = new LoggingProperties()
        {
            LoggingOperations = LoggingOperations.All,
            RetentionDays = 14,
            Version = "1.0"
        }
    });

    // The permissions granted by the account SAS also permit you to retrieve service properties.
    ServiceProperties serviceProperties = blobClientWithSAS.GetServiceProperties();
    Console.WriteLine(serviceProperties.HourMetrics.MetricsLevel);
    Console.WriteLine(serviceProperties.HourMetrics.RetentionDays);
    Console.WriteLine(serviceProperties.HourMetrics.Version);
}
```

### <a name="example-create-a-stored-access-policy"></a>예제: 저장된 액세스 정책 만들기
다음 코드는 컨테이너에 저장된 액세스 정책을 만듭니다. 액세스 정책을 사용하여 컨테이너나 해당 Blob에 서비스 SAS에 대한 제약 조건을 지정할 수 있습니다.

```csharp
private static async Task CreateSharedAccessPolicyAsync(CloudBlobContainer container, string policyName)
{
    // Create a new shared access policy and define its constraints.
    // The access policy provides create, write, read, list, and delete permissions.
    SharedAccessBlobPolicy sharedPolicy = new SharedAccessBlobPolicy()
    {
        // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
        // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
        SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
        Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List |
            SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create | SharedAccessBlobPermissions.Delete
    };

    // Get the container's existing permissions.
    BlobContainerPermissions permissions = await container.GetPermissionsAsync();

    // Add the new policy to the container's permissions, and set the container's permissions.
    permissions.SharedAccessPolicies.Add(policyName, sharedPolicy);
    await container.SetPermissionsAsync(permissions);
}
```

### <a name="example-create-a-service-sas-on-a-container"></a>예제: 컨테이너에 서비스 SAS 만들기
다음 코드는 컨테이너에 SAS를 만듭니다. 기존에 저장된 액세스 정책의 이름을 제공하는 경우 해당 정책은 SAS와 연결됩니다. 저장된 액세스 정책이 제공되지 않는 경우 코드는 컨테이너에서 임시 SAS을 만듭니다.

```csharp
private static string GetContainerSasUri(CloudBlobContainer container, string storedPolicyName = null)
{
    string sasContainerToken;

    // If no stored policy is specified, create a new access policy and define its constraints.
    if (storedPolicyName == null)
    {
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad-hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocPolicy = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.List
        };

        // Generate the shared access signature on the container, setting the constraints directly on the signature.
        sasContainerToken = container.GetSharedAccessSignature(adHocPolicy, null);

        Console.WriteLine("SAS for blob container (ad hoc): {0}", sasContainerToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the container. In this case, all of the constraints for the
        // shared access signature are specified on the stored access policy, which is provided by name.
        // It is also possible to specify some constraints on an ad-hoc SAS and others on the stored access policy.
        sasContainerToken = container.GetSharedAccessSignature(null, storedPolicyName);

        Console.WriteLine("SAS for blob container (stored access policy): {0}", sasContainerToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return container.Uri + sasContainerToken;
}
```

### <a name="example-create-a-service-sas-on-a-blob"></a>예제: Blob에 서비스 SAS 만들기
다음 코드는 Blob에 SAS를 만듭니다. 기존에 저장된 액세스 정책의 이름을 제공하는 경우 해당 정책은 SAS와 연결됩니다. 저장된 액세스 정책이 제공되지 않는 경우 코드는 Blob에서 임시 SAS을 만듭니다.

```csharp
private static string GetBlobSasUri(CloudBlobContainer container, string blobName, string policyName = null)
{
    string sasBlobToken;

    // Get a reference to a blob within the container.
    // Note that the blob may not exist yet, but a SAS can still be created for it.
    CloudBlockBlob blob = container.GetBlockBlobReference(blobName);

    if (policyName == null)
    {
        // Create a new access policy and define its constraints.
        // Note that the SharedAccessBlobPolicy class is used both to define the parameters of an ad-hoc SAS, and
        // to construct a shared access policy that is saved to the container's shared access policies.
        SharedAccessBlobPolicy adHocSAS = new SharedAccessBlobPolicy()
        {
            // When the start time for the SAS is omitted, the start time is assumed to be the time when the storage service receives the request.
            // Omitting the start time for a SAS that is effective immediately helps to avoid clock skew.
            SharedAccessExpiryTime = DateTime.UtcNow.AddHours(24),
            Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.Write | SharedAccessBlobPermissions.Create
        };

        // Generate the shared access signature on the blob, setting the constraints directly on the signature.
        sasBlobToken = blob.GetSharedAccessSignature(adHocSAS);

        Console.WriteLine("SAS for blob (ad hoc): {0}", sasBlobToken);
        Console.WriteLine();
    }
    else
    {
        // Generate the shared access signature on the blob. In this case, all of the constraints for the
        // shared access signature are specified on the container's stored access policy.
        sasBlobToken = blob.GetSharedAccessSignature(null, policyName);

        Console.WriteLine("SAS for blob (stored access policy): {0}", sasBlobToken);
        Console.WriteLine();
    }

    // Return the URI string for the container, including the SAS token.
    return blob.Uri + sasBlobToken;
}
```

## <a name="conclusion"></a>결론
공유 액세스 서명은 계정 키가 필요하지 않은 클라이언트에게 저장소 계정에 대한 제한된 권한을 제공하는 데 유용합니다. 일반적으로 공유 액세스 서명은 Azure Storage를 사용하는 응용 프로그램에 대한 보안 모델의 필수적인 부분입니다. 여기에 나열된 모범 사례를 따를 경우 SAS를 사용하여 응용 프로그램의 보안을 훼손하지 않으면서 저장소 계정에 있는 리소스에 유연하게 액세스할 수 있습니다.

## <a name="next-steps"></a>다음 단계
* [공유 액세스 서명, 2부: Blob 저장소를 사용하여 SAS 만들기 및 사용](../blobs/storage-dotnet-shared-access-signature-part-2.md)
* [컨테이너 및 Blob에 대한 익명 읽기 권한 관리](../blobs/storage-manage-access-to-resources.md)
* [공유 액세스 서명을 사용하여 액세스 위임](http://msdn.microsoft.com/library/azure/ee395415.aspx)
* [테이블 및 큐 SAS 소개](http://blogs.msdn.com/b/windowsazurestorage/archive/2012/06/12/introducing-table-sas-shared-access-signature-queue-sas-and-update-to-blob-sas.aspx)
