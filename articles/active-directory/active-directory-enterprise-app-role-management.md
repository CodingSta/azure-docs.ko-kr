---
title: Azure Active Directory의 엔터프라이즈 응용 프로그램에 대해 SAML 토큰에서 발급된 역할 클레임 구성 | Microsoft Docs
description: Azure Active Directory의 엔터프라이즈 응용 프로그램에 SAML 토큰에서 발급된 역할 클레임을 구성하는 방법 알아보기
services: active-directory
documentationcenter: ''
author: jeevansd
manager: mtillman
editor: ''
ms.assetid: eb2b3741-3cde-45c8-b639-a636f3df3b74
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/20/2018
ms.author: jeedes
ms.custom: aaddev
ms.openlocfilehash: 58656e2aa3b052d9bd9aa14edeb6215858d55ea4
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2018
---
# <a name="configuring-role-claim-issued-in-the-saml-token-for-enterprise-applications-in-azure-active-directory"></a>Azure Active Directory의 엔터프라이즈 응용 프로그램에 대해 SAML 토큰에서 발급된 역할 클레임 구성

이 기능을 사용하면 사용자는 Azure AD를 사용하여 앱에 권한을 부여할 때 받은 응답 토큰의 '역할' 클레임에 대한 클레임 형식을 사용자 지정할 수 있습니다.

## <a name="prerequisites"></a>필수 조건
- 디렉터리가 설치된 Azure AD 구독
- Single Sign-on이 설정된 구독
- 응용 프로그램에 SSO를 구성해야 함

## <a name="when-to-use-this-feature"></a>이 기능을 사용하는 시기

응용 프로그램에서 사용자 지정 역할이 SAML 응답에 전달될 것으로 예상하는 경우 이 기능을 사용해야 합니다. 이 기능을 사용하면 Azure AD에서 다시 응용 프로그램으로 전달하는 데 필요한 만큼 역할을 만들 수 있습니다.

## <a name="steps-to-use-this-feature"></a>이 기능을 사용하는 단계

1. **[Azure Portal](https://portal.azure.com)**의 왼쪽 탐색 창에서 **Azure Active Directory** 아이콘을 클릭합니다. 

    ![Azure Active Directory 단추][1]

2. **엔터프라이즈 응용 프로그램**으로 이동합니다. 그런 후 **모든 응용 프로그램**으로 이동합니다.

    ![엔터프라이즈 응용 프로그램 블레이드][2]
    
3. 새 응용 프로그램을 추가하려면 대화 상자 맨 위 있는 **새 응용 프로그램** 단추를 클릭합니다.

    ![새 응용 프로그램 단추][3]

4. 검색 상자에 응용 프로그램 이름을 입력하고, 결과 패널에서 응용 프로그램을 선택한 후 **추가** 단추를 클릭하여 응용 프로그램을 추가합니다.

    ![결과 목록의 응용 프로그램](./media/active-directory-enterprise-app-role-management/tutorial_app_addfromgallery.png)

5. 응용 프로그램이 추가되면 **속성** 페이지로 이동하여 **개체 ID**를 복사합니다.

    ![속성 페이지](./media/active-directory-enterprise-app-role-management/tutorial_app_properties.png)

6. 또 다른 창에서 [Azure AD Graph Explorer](https://developer.microsoft.com/graph/graph-explorer)를 엽니다.

    a. 테넌트의 전역 관리자/공동 관리자 자격 증명을 사용하여 Graph Explorer 사이트에 로그인합니다.

    나. 버전을 **베타**로 변경하고 다음 쿼리를 사용하여 테넌트에서 서비스 사용자 목록을 가져옵니다.
    
     `https://graph.microsoft.com/beta/servicePrincipals`
        
    여러 디렉터리를 사용하는 경우 `https://graph.microsoft.com/beta/contoso.com/servicePrincipals` 패턴을 따라야 합니다.
    
    ![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer1-updated.png)
    
    다. 가져온 서비스 사용자 목록에서 수정할 서비스 사용자를 가져옵니다. Ctrl+F를 사용하여 나열된 모든 ServicePrincipals에서 응용 프로그램을 검색할 수도 있습니다. [속성] 페이지에서 복사한 **개체 ID**를 검색하고 다음 쿼리를 사용하여 각 서비스 사용자로 이동합니다.
    
    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`

    d. 서비스 사용자 개체에서 appRoles 속성을 추출합니다.

    ![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-approles.png)

    e. 이제 응용 프로그램에 대한 새 역할을 생성해야 합니다. Azure AD 역할 생성기는 [여기](https://app.box.com/s/jw6m9p9ehmf4ut5jx8xhcw87cu09ml3y)서 다운로드할 수 있습니다.

    f. Azure AD 생성기를 열고 다음 단계를 수행합니다.

    ![Azure AD 생성기](./media/active-directory-enterprise-app-role-management/azure_ad_role_generator.png)
    
    **역할 이름**, **역할 설명** 및 **역할 값**을 입력합니다. **추가**를 클릭하여 역할을 추가합니다.
    
    필요한 역할을 모두 추가한 후 **생성**을 클릭합니다.
    
    **콘텐츠 복사**를 클릭하여 콘텐츠를 복사합니다.

    > [!NOTE] 
    > 생성된 역할에서 **msiam_access** 사용자 역할과 id가 일치하는지 확인합니다.

    g. Graph Explorer로 돌아갑니다. 메서드를 **GET**에서 **PATCH**로 변경합니다. 복사한 값으로 appRoles 속성을 업데이트하여 원하는 appRoles를 사용하도록 서비스 사용자 개체를 패치합니다. **쿼리 실행**을 클릭합니다.

    ![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-patch.png)

    > [!NOTE]
    > 다음은 appRoles 개체의 예입니다. 
    ```
    {
       "appRoles": [
    {
        "allowedMemberTypes": [
        "User"
        ],
        "description": "msiam_access",
        "displayName": "msiam_access",
        "id": "7dfd756e-8c27-4472-b2b7-38c17fc5de5e",
        "isEnabled": true,
        "origin": "Application",
        "value": null
    },
    {
        "allowedMemberTypes": [
        "User"
        ],
        "description": "teacher",
        "displayName": "teacher",
        "id": "6478ffd2-5dbd-4584-b2ce-137390b09b60",
        "isEnabled": ,
        "origin": "ServicePrincipal",
        "value": "teacher"
    }
    ] 
    }   
    ```
7. 서비스 사용자에 더 많은 역할이 패치되면 각 역할에 사용자를 할당할 수 있습니다. 이 작업을 수행하려면 포털로 이동한 후 각 앱으로 이동합니다. 그런 다음, 맨 위에서 **사용자 및 그룹** 탭을 클릭합니다. 이 프로세스는 모든 사용자 또는 그룹을 나열합니다.

    ![Single Sign-On 구성 추가](./media/active-directory-enterprise-app-role-management/userrole.png)

    a. 사용자에게 역할을 할당하려면 특정 사용자/그룹을 선택하고 페이지 아래쪽에서 **할당** 단추를 클릭합니다.

    ![Single Sign-On 구성 추가](./media/active-directory-enterprise-app-role-management/userandgroups.png)

    나. 이 단추를 클릭하면 각 서비스 사용자에 대해 정의된 여러 역할 중에서 선택하는 팝업이 나타납니다.

    다. 필요한 역할을 선택하고 [제출]을 클릭합니다.

8. 사용자에게 역할을 할당한 후에는 **역할** 클레임의 사용자 지정 매핑을 정의하도록 **특성** 테이블을 업데이트해야 합니다.

9. **Single Sign-On** 대화 상자의 **사용자 특성** 섹션에서 이미지에 표시된 것과 같이 SAML 토큰 특성을 구성하고 다음 단계를 수행합니다.
    
    | 특성 이름 | 특성 값 |
    | -------------- | ----------------|    
    | 역할 이름      | user.assignedrole |

    a. **특성 추가**를 클릭하여 **특성 추가** 대화 상자를 엽니다.

    ![Single Sign-On 구성 추가](./media/active-directory-enterprise-app-role-management/tutorial_attribute_04.png)

    ![Single Sign-On 특성 구성](./media/active-directory-enterprise-app-role-management/tutorial_attribute_05.png)

    나. **이름** 텍스트 상자에서 해당 행에 표시된 특성 이름을 입력합니다.

    다. **값** 목록에서 해당 행에 대해 표시된 특성을 입력합니다.

    d. **네임스페이스**를 비워 둡니다.
    
    e. **Ok**를 클릭합니다.

10. IDP 시작 Single Sign On에서 응용 프로그램을 테스트하려면 액세스 패널에 로그인(https://myapps.microsoft.com)하여 응용 프로그램 타일을 클릭합니다. SAML 토큰을 보면 지정한 클레임 이름의 사용자에게 할당된 모든 역할이 표시됩니다.

## <a name="update-existing-role"></a>기존 역할 업데이트

1. 기존 역할을 업데이트하려면 다음 단계를 수행합니다.

    a. 또 다른 창에서 [Azure AD Graph Explorer](https://developer.microsoft.com/graph/graph-explorer)를 엽니다.

    나. 테넌트의 전역 관리자/공동 관리자 자격 증명을 사용하여 Graph Explorer 사이트에 로그인합니다.
    
    다. 버전을 **베타**로 변경하고 다음 쿼리를 사용하여 테넌트에서 서비스 사용자 목록을 가져옵니다.
    
    `https://graph.microsoft.com/beta/servicePrincipals`
        
    여러 디렉터리를 사용하는 경우 `https://graph.microsoft.com/beta/contoso.com/servicePrincipals` 패턴을 따라야 합니다.
    
    ![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer1-updated.png)
    
    d. 가져온 서비스 사용자 목록에서 수정할 서비스 사용자를 가져옵니다. Ctrl+F를 사용하여 나열된 모든 ServicePrincipals에서 응용 프로그램을 검색할 수도 있습니다. [속성] 페이지에서 복사한 **개체 ID**를 검색하고 다음 쿼리를 사용하여 각 서비스 사용자로 이동합니다.
    
    `https://graph.microsoft.com/beta/servicePrincipals/<objectID>`
    
    e. 서비스 사용자 개체에서 appRoles 속성을 추출합니다.
    
    ![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-approles.png)
    
    f. 기존 역할을 업데이트하려면 다음 단계를 수행합니다.

    ![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-patchupdate.png)
    
    * 메서드를 **GET**에서 **PATCH**로 변경합니다.

    * 응용 프로그램에서 기존 역할을 복사하여 **요청 본문**에 붙여넣습니다.
    
    * 조직 요구 사항에 따라 **역할 설명**, **역할 값** 및 **역할 표시 이름**을 바꿔서 값을 업데이트합니다.
    
    * 필요한 역할을 모두 업데이트한 후 **쿼리 실행**을 클릭합니다.
        
## <a name="delete-existing-role"></a>기존 역할 삭제

기존 역할을 삭제하려면 다음 단계를 수행합니다.

a. 또 다른 창에서 [Azure AD Graph Explorer](https://developer.microsoft.com/graph/graph-explorer)를 엽니다.

나. 테넌트의 전역 관리자/공동 관리자 자격 증명을 사용하여 Graph Explorer 사이트에 로그인합니다.

다. 버전을 **베타**로 변경하고 다음 쿼리를 사용하여 테넌트에서 서비스 사용자 목록을 가져옵니다.
    
`https://graph.microsoft.com/beta/servicePrincipals`
    
여러 디렉터리를 사용하는 경우 `https://graph.microsoft.com/beta/contoso.com/servicePrincipals` 패턴을 따라야 합니다.
    
![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer1-updated.png)
    
d. 가져온 서비스 사용자 목록에서 수정할 서비스 사용자를 가져옵니다. Ctrl+F를 사용하여 나열된 모든 ServicePrincipals에서 응용 프로그램을 검색할 수도 있습니다. [속성] 페이지에서 복사한 **개체 ID**를 검색하고 다음 쿼리를 사용하여 각 서비스 사용자로 이동합니다.
     
`https://graph.microsoft.com/beta/servicePrincipals/<objectID>`
    
e. 서비스 사용자 개체에서 appRoles 속성을 추출합니다.
    
![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-approles.png)

f. 기존 역할을 삭제하려면 다음 단계를 수행합니다.

![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-patchdelete.png)

메서드를 **GET**에서 **PATCH**로 변경합니다.

응용 프로그램에서 기존 역할을 복사하여 **요청 본문**에 붙여넣습니다.
    
삭제하려는 역할의 **IsEnabled** 값을 **false**로 설정합니다.

**쿼리 실행**을 클릭합니다.
    
> [!NOTE] 
> 생성된 역할에서 **msiam_access** 사용자 역할과 id가 일치하는지 확인합니다.
    
g. 위 프로세스를 수행한 후 메서드를 **PATCH**로 유지하고 나머지 역할 콘텐츠를 **요청 본문**에 붙여넣은 후 **쿼리 실행**을 클릭합니다.
    
![Graph Explorer 대화 상자](./media/active-directory-enterprise-app-role-management/graph-explorer-patchfinal.png)

h. 쿼리를 실행한 후에는 역할이 삭제됩니다.
    
> [!NOTE]
> 역할을 제거하려면 먼저 비활성화해야 합니다. 

## <a name="next-steps"></a>다음 단계

[앱 설명서](https://docs.microsoft.com/en-us/azure/active-directory/active-directory-saas-tutorial-list)에서 추가 단계를 참조하세요.

<!--Image references-->
<!--Image references-->

[1]: ./media/active-directory-enterprise-app-role-management/tutorial_general_01.png
[2]: ./media/active-directory-enterprise-app-role-management/tutorial_general_02.png
[3]: ./media/active-directory-enterprise-app-role-management/tutorial_general_03.png
[4]: ./media/active-directory-enterprise-app-role-management/tutorial_general_04.png
