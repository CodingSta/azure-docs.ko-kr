---
title: Azure Data Factory에서 Hadoop Hive 작업을 사용하여 데이터 변환 | Microsoft Docs
description: Azure 데이터 공장에서 Hive 활동을 사용하여 필요 시/사용자 고유의 HDInsight 클러스터에서 Hive 쿼리를 실행하는 방법을 알아봅니다.
services: data-factory
documentationcenter: ''
author: shengcmsft
manager: craigg
ms.reviewer: douglasl
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/16/2018
ms.author: shengc
ms.openlocfilehash: 637a9ce68cc1c8ac2ef0af8a606668fb4da64fb8
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: ko-KR
ms.lasthandoff: 03/23/2018
---
# <a name="transform-data-using-hadoop-hive-activity-in-azure-data-factory"></a>Azure Data Factory에서 Hadoop Hive 작업을 사용하여 데이터 변환
> [!div class="op_single_selector" title1="Select the version of Data Factory service you are using:"]
> * [버전 1 - GA](v1/data-factory-hive-activity.md)
> * [버전 2 - 미리 보기](transform-data-using-hadoop-hive.md)

Data Factory [파이프라인](concepts-pipelines-activities.md)의 HDInsight Hive 작업은 [사용자 고유](compute-linked-services.md#azure-hdinsight-linked-service) 또는 [주문형](compute-linked-services.md#azure-hdinsight-on-demand-linked-service) HDInsight 클러스터의 Hive 쿼리를 실행합니다. 이 문서는 데이터 변환 및 지원되는 변환 활동의 일반적인 개요를 표시하는 [데이터 변환 활동](transform-data.md) 문서에서 작성합니다.

> [!NOTE]
> 이 문서는 현재 미리 보기 상태인 Data Factory 버전 2에 적용됩니다. GA(일반 공급) 상태인 Data Factory 버전 1 서비스를 사용 중인 경우 [V1의 Hive 작업](v1/data-factory-hive-activity.md)을 참조하세요.

Azure Data Factory를 처음 접하는 경우 [Azure Data Factory 소개](introduction.md)를 읽고 이 문서를 읽기 전에 [자습서: 데이터 변환](tutorial-transform-data-spark-powershell.md)을 수행하세요. 

## <a name="syntax"></a>구문

```json
{
    "name": "Hive Activity",
    "description": "description",
    "type": "HDInsightHive",
    "linkedServiceName": {
        "referenceName": "MyHDInsightLinkedService",
        "type": "LinkedServiceReference"
    },
    "typeProperties": {
        "scriptLinkedService": {
            "referenceName": "MyAzureStorageLinkedService",
            "type": "LinkedServiceReference"
        },
        "scriptPath": "MyAzureStorage\\HiveScripts\\MyHiveSript.hql",
        "getDebugInfo": "Failure",
        "arguments": [
            "SampleHadoopJobArgument1"
        ],
        "defines": {
            "param1": "param1Value"
        }
    }   
}
```
## <a name="syntax-details"></a>구문 세부 정보
| 자산            | 설명                              | 필수 |
| ------------------- | ---------------------------------------- | -------- |
| 이름                | 작업의 이름                     | 예      |
| description         | 작업이 무엇에 사용되는지 설명하는 텍스트입니다. | 아니오       |
| 형식                | Hive 작업의 경우 작업 유형은 HDinsightHive입니다. | 예      |
| linkedServiceName   | Data Factory에서 연결된 서비스로 등록된 HDInsight 클러스터에 대한 참조입니다. 이 연결된 서비스에 대한 자세한 내용은 [연결된 Compute Services](compute-linked-services.md) 문서를 참조하세요. | 예      |
| scriptLinkedService | 실행할 Hive 스크립트를 저장하는 데 사용되는 Azure Storage 연결된 서비스에 대한 참조입니다. 이 연결된 서비스를 지정하지 않으면 HDInsight 연결된 서비스에 정의된 Azure Storage 연결된 서비스가 사용됩니다. | 아니오       |
| scriptPath          | scriptLinkedService에서 참조하는 Azure Storage에 저장된 스크립트 파일의 경로를 제공합니다. 파일 이름은 대/소문자를 구분합니다. | 예      |
| getDebugInfo        | scriptLinkedService에 지정되었거나 HDInsight 클러스터에 사용된 Azure Storage에 로그 파일을 언제 복사할지 지정합니다. 허용되는 값: None, Always 또는 Failure. 기본값: None. | 아니요       |
| arguments           | Hadoop 작업에 대한 인수 배열을 지정합니다. 인수는 각 작업에 대한 명령줄 인수로 전달됩니다. | 아니오       |
| defines             | Hive 스크립트 내에서 참조하기 위해 매개 변수를 키/값 쌍으로 지정합니다. | 아니오       |

## <a name="next-steps"></a>다음 단계
다른 방법으로 데이터를 변환하는 방법을 설명하는 다음 문서를 참조하세요. 

* [U-SQL 작업](transform-data-using-data-lake-analytics.md)
* [Pig 작업](transform-data-using-hadoop-pig.md)
* [MapReduce 작업](transform-data-using-hadoop-map-reduce.md)
* [Hadoop 스트리밍 작업](transform-data-using-hadoop-streaming.md)
* [Spark 작업](transform-data-using-spark.md)
* [.NET 사용자 지정 작업](transform-data-using-dotnet-custom-activity.md)
* [Machine Learning Batch 실행 작업](transform-data-using-machine-learning.md)
* [저장 프로시저 작업](transform-data-using-stored-procedure.md)

