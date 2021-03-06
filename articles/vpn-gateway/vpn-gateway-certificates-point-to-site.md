---
title: "使用 makecert 建立點對站虛擬網路交叉部署連線的自我簽署憑證 |Microsoft Docs"
description: "本文包含使用 makecert 在 Windows 10 上建立自我簽署憑證的步驟。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: carmonm
editor: 
tags: azure-resource-manager
ms.assetid: 27b99f7c-50dc-4f88-8a6e-d60080819a43
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/22/2016
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: b44f46273c6ddb439ec3793f193986011c222ee3


---
# <a name="working-with-self-signed-certificates-for-point-to-site-connections"></a>使用點對站連線的自我簽署憑證
本文將協助您使用 **makecert**建立自我簽署憑證，然後再從它產生用戶端憑證。 下列步驟專為 Windows 10 上的 makecert 所撰寫。 Makecert 已經驗證以建立與 P2S 連線相容的憑證。 

對於 P2S 連線，針對憑證的慣用方法是使用您的企業憑證解決方案，確保使用具有通用名稱值格式「'name@yourdomain.com',」核發用戶端憑證，而非使用「NetBIOS 網域名稱\使用者名稱」格式。

如果您沒有企業解決方案，則需要自我簽署憑證以允許 P2S 用戶端連接到虛擬網路。 我們知道 makecert 已經過時，但它仍是建立與 P2S 連線相容的自我簽署憑證的有效方法。 我們正在研究建立自我簽署憑證的另一個解決方案，但就目前而言，makecert 是慣用的方法。

## <a name="create-a-self-signed-certificate"></a>建立自我簽署憑證
Makecert 是建立自我簽署憑證的其中一種方式。 下列步驟將逐步引導您使用 makecert 建立自我簽署憑證。 這些並非部署模型特定的步驟。 它們同樣適用於資源管理員和傳統部署模型。

### <a name="to-create-a-self-signed-certificate"></a>建立自我簽署憑證
1. 從執行 Windows 10 的電腦上下載並安裝[適用於 Windows 10 的 Windows 軟體開發套件 (SDK)](https://dev.windows.com/en-us/downloads/windows-10-sdk)。
2. 安裝之後，您可以在下列路徑中找到 makecert.exe 公用程式：C:\Program Files (x86)\Windows Kits\10\bin\<arch>。 
   
    範例： `C:\Program Files (x86)\Windows Kits\10\bin\x64`
3. 接下來，在您電腦上的個人憑證存放區中建立並安裝憑證。 下列範例會建立對應的 .cer 檔案，您在設定 P2S 時會將此檔案上傳至 Azure。 以系統管理員身分執行下列命令。 將 ARMP2SRootCert 和 ARMP2SRootCert.cer 取代為您想使用的憑證名稱。<br><br>憑證會位於您的憑證 - 目前使用者\個人\憑證。
   
        makecert -sky exchange -r -n "CN=ARMP2SRootCert" -pe -a sha1 -len 2048 -ss My "ARMP2SRootCert.cer"

### <a name="a-namerootpublickeyato-obtain-the-public-key"></a><a name="rootpublickey"></a>取得公開金鑰
做為點對站連線的 VPN 閘道組態的一部分，根憑證的公開金鑰會上傳至 Azure。

1. 若要取得憑證的 .cer 檔案，請開啟 [certmgr.msc] 。 在自我簽署的根憑證上按一下滑鼠右鍵，按一下 [所有工作]，然後按一下 [匯出]。 這會開啟 [憑證匯出精靈] 。
2. 在精靈中，按 [下一步]，接著選取 [否，不要匯出私密金鑰]，然後按 [下一步]。
3. 在 [匯出檔案格式] 頁面上，選取 [Base-64 編碼 X.509 (.CER)]。 然後按 [下一步] 。 
4. 在 [要匯出的檔案] 中，[瀏覽] 到您要匯出憑證的位置。 針對 [檔案名稱] ，請為憑證檔案命名。 然後按 [下一步] 。
5. 按一下 [完成]  以匯出憑證。

### <a name="export-the-self-signed-certificate-optional"></a>匯出自我簽署的憑證 (選擇性)
您可能想要匯出自我簽署的憑證，並將它安全地儲存。 如有需要，您可以稍後在另一部電腦上安裝這個自我簽署憑證，然後產生更多用戶端憑證，或匯出另一個 .cer 檔案。 已安裝用戶端憑證並設定適當的 VPN 用戶端設定的任何電腦，都可以透過 P2S 連線到您的虛擬網路。 基於這個原因，您必須確定用戶端憑證只有在需要時才會產生並安裝，而且此自我簽署憑證已安全地儲存。

若要將自我簽署的憑證匯出為 .pfx，請選取根憑證，並使用 [匯出用戶端憑證](#clientkey) 中所述的步驟匯出。

## <a name="create-and-install-client-certificates"></a>建立並安裝用戶端憑證
您未直接在用戶端電腦上安裝自我簽署的憑證。 您需要從自我簽署憑證產生用戶端憑證。 您接著會將用戶端憑證匯出並安裝到用戶端電腦。 下列步驟並非針對特定部署模型。 它們同樣適用於資源管理員和傳統部署模型。

### <a name="part-1---generate-a-client-certificate-from-a-self-signed-certificate"></a>第 1 部分 - 從自我簽署憑證產生用戶端憑證
下列步驟將逐步引導您完成從自我簽署憑證產生用戶端憑證的其中一種方法。 您也可以從相同憑證產生多個用戶端憑證。 然後每個用戶端憑證可以匯出及安裝在用戶端電腦上。 

1. 在用來建立自我簽署憑證的相同電腦上，以系統管理員身分開啟命令提示字元。
2. 在此範例中，"ARMP2SRootCert" 是指您產生的自我簽署憑證。 
   
   * 將 *"ARMP2SRootCert"* 變更為您產生用戶端憑證所用的自我簽署根憑證。 
   * 將 *ClientCertificateName* 變更為產生用戶端憑證所用的名稱。 

    修改並執行範例以產生用戶端憑證。 如果您未經修改就執行以下範例，您的個人憑證存放區中就會有一個從根憑證 ARMP2SRootCert 產生的用戶端憑證，名為 ClientCertificateName。

        makecert.exe -n "CN=ClientCertificateName" -pe -sky exchange -m 96 -ss My -in "ARMP2SRootCert" -is my -a sha1

1. 所有憑證都會儲存在電腦上的憑證 - 目前使用者\個人\憑證存放區中。 您可以視需要根據這個程序來產生許多用戶端憑證。

### <a name="a-nameclientkeyapart-2---export-a-client-certificate"></a><a name="clientkey"></a>第 2 部分 - 匯出用戶端憑證
1. 若要匯出用戶端憑證，請開啟 **certmgr.msc**。 以滑鼠右鍵按一下要匯出的用戶端憑證，然後依序按一下 [所有工作] 和 [匯出]。 這會開啟 [憑證匯出精靈] 。
2. 在精靈中，按一下 [下一步]，接著選取 [是，匯出私密金鑰]，然後按 [下一步]。
3. 在 [匯出檔案格式]  頁面上，您可以保留選取預設值。 然後按 [下一步] 。 
4. 在 [安全性]  頁面上，您必須保護私密金鑰。 如果您選取要使用密碼，請務必記錄或牢記您為此憑證設定的密碼。 然後按 [下一步] 。
5. 在 [要匯出的檔案] 中，[瀏覽] 到您要匯出憑證的位置。 針對 [檔案名稱] ，請為憑證檔案命名。 然後按 [下一步] 。
6. 按一下 [完成]  以匯出憑證。    

### <a name="part-3---install-a-client-certificate"></a>第 3 部分 - 安裝用戶端憑證
您想要使用點對站連線與虛擬網路連線的每個用戶端，都必須安裝用戶端憑證。 此憑證是必要的 VPN 設定封裝以外的項目。 下列步驟將逐步引導您手動安裝用戶端憑證。

1. 找出 *.pfx* 檔案並複製到用戶端電腦。 在用戶端電腦上，按兩下 *.pfx* 檔案以安裝。 [存放區位置] 保留 [目前使用者]，然後按 [下一步]。
2. 在 [要匯入的檔案]  頁面上，請勿進行任何變更。 按一下頁面底部的 [新增] 來單一登入應用程式。
3. 在 [私密金鑰保護] 頁面上，如果您已使用密碼，請輸入憑證的密碼，或確認安裝憑證的安全性主體是否正確，然後按 [下一步]。
4. 在 [憑證存放區] 頁面上，保留預設的位置，然後按 [下一步]。
5. 按一下 [完成] 。 在憑證安裝的 [安全性警告] 上，按一下 [是]。 現在已成功匯入憑證。

## <a name="next-steps"></a>後續步驟
繼續使用您的點對站設定。 

* 針對 **Resource Manager** 部署模型步驟，請參閱 [使用 PowerShell 設定 VNet 的點對站連線](vpn-gateway-howto-point-to-site-rm-ps.md)。 
* 針對「傳統」  部署模型步驟，請參閱 [使用傳統入口網站設定 VNet 的點對站 VPN 連線](vpn-gateway-point-to-site-create.md)。




<!--HONumber=Nov16_HO3-->


