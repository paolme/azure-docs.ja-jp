---
title: 'クイック スタート: Azure portal を使用してサインイン レポートをダウンロードする | Microsoft Docs'
description: Azure portal を使用してサインイン レポートをダウンロードする方法について説明します
services: active-directory
documentationcenter: ''
author: priyamohanram
manager: mtillman
editor: ''
ms.assetid: 9131f208-1f90-4cc1-9c29-085cacd69317
ms.service: active-directory
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.component: report-monitor
ms.date: 11/13/2018
ms.author: priyamo
ms.reviewer: dhanyahk
ms.openlocfilehash: 0e6e72424530d18b55f68077ba7c3328d9a2e549
ms.sourcegitcommit: 1f9e1c563245f2a6dcc40ff398d20510dd88fd92
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 11/14/2018
ms.locfileid: "51621430"
---
# <a name="quickstart-download-a-sign-in-report-using-the-azure-portal"></a>クイック スタート: Azure portal を使用してサインイン レポートをダウンロードする

このクイック スタートでは、過去 24 時間のテナントに関するサインイン データをダウンロードする方法について説明します。

## <a name="prerequisites"></a>前提条件

必要なもの:

* Premium ライセンスを使用してサインイン アクティビティ レポートを表示する Azure Active Directory テナント。 
* テナントの**セキュリティ管理者**、**セキュリティ閲覧者**、**レポート閲覧者**、**グローバル管理者**のいずれかのロールであるユーザー。 さらに、テナントのすべてのユーザーは自分のサインインにアクセスできます。

## <a name="quickstart-download-a-sign-in-report"></a>クイック スタート: サインイン レポートをダウンロードする

1. [Azure Portal](https://portal.azure.com) に移動します。
2. 左側のナビゲーション ウィンドウで **[Azure Active Directory]** を選択し、**[ディレクトリの切り替え]** ボタンを使用して目的の Active Directory を選択します。
3. ダッシュボードから **[Azure Active Directory]** を選択し、**[サインイン]** を選択します。 
4. **[日付]** フィルター ドロップダウンで **[過去 24 時間]** を選択し、**[適用]** を選択して過去 24 時間のサインインを表示します。 
5. **[ダウンロード]** ボタンを選択して、フィルター処理したレコードを含む CSV ファイルをダウンロードします。 

![レポート](./media/quickstart-download-sign-in-report/download-sign-ins.png)

## <a name="next-steps"></a>次の手順

* [Azure Active Directory ポータルのサインイン アクティビティ レポート](concept-sign-ins.md)
* [Azure Active Directory レポートの保持期間](reference-reports-data-retention.md)
* [Azure Active Directory レポートの待機時間](reference-reports-latencies.md)
