---
title: "針對 Azure 檔案儲存體的問題進行疑難排解 | Microsoft Docs"
description: "針對 Azure 檔案儲存體的問題進行疑難排解"
services: storage
documentationcenter: 
author: genlin
manager: felixwu
editor: na
tags: storage
ms.assetid: fbc5f600-131e-4b99-828a-42d0de85fff7
ms.service: storage
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/13/2016
ms.author: genli
translationtype: Human Translation
ms.sourcegitcommit: afce238686f5b35a094f0792f8197b686d317fa5
ms.openlocfilehash: b75e80b66e00be0022102c06113eb414f5b4b6e9


---
# <a name="troubleshooting-azure-file-storage-problems"></a>針對 Azure 檔案儲存體的問題進行疑難排解
本文列出當您從 Windows 和 Linux 用戶端連接到 Microsoft Azure 檔案儲存體時相關的常見問題。 文中也會提供這些問題的可能原因和解決辦法。

**一般問題 (發生在 Windows 和 Linux 用戶端)**

* [嘗試開啟檔案時，發生配額錯誤](#quotaerror)
* [當您從 Windows 或 Linux 存取 Azure 檔案儲存體時效能緩慢](#slowboth)

**Windows 用戶端的問題**

* [當您從 Windows 8.1 或 Windows Server 2012 R2 存取 Azure 檔案儲存體時效能緩慢](#windowsslow)
* [錯誤 53 嘗試掛接 Azure 檔案共用](#error53)
* [net use 成功，但在 Windows 檔案總管中看不到掛接的 Azure 檔案共用](#netuse)
* [我的儲存體帳戶包含 "/"，net use 命令失敗](#slashfails)
* [我的應用程式/服務無法存取掛接的 Azure 檔案磁碟機](#accessfiledrive)
* [效能最佳化的其他建議](#additional)

**Linux 用戶端的問題**

* [將檔案上傳/複製至 Azure 檔案時，發生「您正將檔案複製到不支援加密的目的地」錯誤](#encryption)
* [在現有檔案共用上發生「主機當機」錯誤，或在掛接點上進行清單命令時殼層停止回應](#errorhold)
* [嘗試在 Linux VM 上掛接 Azure 檔案時，發生掛接錯誤 115](#error15)
* [使用類似 "ls" 的命令時 Linux VM 遇到隨機延遲](#delayproblem)

<a id="quotaerror"></a>

## <a name="quota-error-when-trying-to-open-a-file"></a>嘗試開啟檔案時，發生配額錯誤
在 Windows 中，您會收到類似以下的錯誤訊息︰

**1816 ERROR_NOT_ENOUGH_QUOTA <--> 0xc0000044**

**STATUS_QUOTA_EXCEEDED**

**Not enough quota is available to process this command**

**Invalid handle value GetLastError: 53**

在 Linux 上，您會收到類似以下的錯誤訊息︰

**<filename> [permission denied]**

**Disk quota exceeded**

### <a name="cause"></a>原因
因為您已達到檔案所允許的同時開啟控點上限，就會發生此問題。

### <a name="solution"></a>方案
關閉一些控點以減少同時開啟的控點數，並再試一次。 如需詳細資訊，請參閱 [Microsoft Azure 儲存體效能與延展性檢查清單](storage-performance-checklist.md)。

<a id="slowboth"></a>

## <a name="slow-performance-when-accessing-file-storage-from-windows-or-linux"></a>從 Windows 或 Linux 存取 Azure 檔案儲存體時效能緩慢
* 如果您沒有特定的 I/O 大小需求下限，建議您使用 1 MB 的 I/O 大小以獲得最佳效能。
* 如果您知道擴充寫入檔案的最終大小，而且當檔案上尚未寫入的結尾中有零時您的軟體沒有相容性問題，則請將事先設定檔案大小，而不是在每次寫入是擴充寫入時設定。

<a id="windowsslow"></a>

## <a name="slow-performance-when-accessing-the-file-storage-from-windows-81-or-windows-server-2012-r2"></a>從 Windows 8.1 或 Windows Server 2012 R2 存取檔案儲存體時效能緩慢
若用戶端執行 Windows 8.1 或 Windows Server 2012 R2，請確定已安裝 hotfix [KB3114025](https://support.microsoft.com/kb/3114025)。 此 hotfix 可改善建立和關閉控點的效能。

您可以執行下列指令碼來檢查是否已安裝此 hotfix：

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`

如果已安裝 hotfix，則會顯示下列輸出︰

**HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies**

**{96c345ef-3cac-477b-8fcd-bea1a564241c}    REG_DWORD    0x1**

> [!NOTE]
> 自 2015 年 12 月起，Azure Marketplace 中的 Windows Server 2012 R2 映像預設已安裝 hotfix KB3114025。
>
>

<a id="additional"></a>

## <a name="additional-recommendations-to-optimize-performance"></a>效能最佳化的其他建議
絕對不要建立或開啟檔案進行要求寫入存取但不要求讀取存取的快取 I/O。 也就是說，當您呼叫 **Createfile()**，絕對不要只指定 **GENERIC_WRITE**，一定要指定 **GENERIC_READ |GENERIC_WRITE**。 唯寫的控點無法在本機快取少量寫入，即使它是該檔案的唯一開啟控點。 這會在少量寫入時帶來嚴重的效能損失。 請注意，"a" 模式 CRT **fopen()** 會開啟唯寫的控點。

<a id="error53"></a>

## <a name="error-53-when-you-try-to-mount-or-unmount-an-azure-file-share"></a>當您嘗試掛接或取消掛接 Azure 檔案共用時發生「錯誤 53」
這個問題可能是因下列情況而起︰

### <a name="cause-1"></a>原因 1
「發生系統錯誤 53。 存取遭到拒絕。」 基於安全性理由，如果通訊通道沒有加密，Azure 檔案共用的連線會遭到封鎖，故不會從 Azure 檔案共用所在的相同資料中心進行連線嘗試。 如果使用者的用戶端作業系統不支援 SMB 加密，則不會提供通訊通道加密。 指出這個狀況的就是「發生系統錯誤 53。 存取遭到拒絕。」錯誤訊息，在使用者嘗試從內部部署或不同的資料中心掛接檔案共用時出現。 Windows 8、Windows Server 2012 和更新版本的每個交涉皆會要求包含支援加密的 SMB 3.0。

### <a name="solution-for-cause-1"></a>原因 1 的解決辦法
從符合 Windows 8、Windows Server 2012 和更新版本需求的用戶端進行連線，或是從 Azure 檔案共用所使用的 Azure 儲存體帳戶相同的資料中心上的虛擬機器進行連線。

### <a name="cause-2"></a>原因 2
當您掛接 Azure 檔案共用時，如果連接埠 445 至 Azure 檔案資料中心的輸出通訊遭到封鎖，可能會發生「系統錯誤 53」。 按一下[這裡](http://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx)查看 ISP 是否允許從連接埠 445 進行存取的摘要。

Comcast 和某些 IT 組織會封鎖此連接埠。 若要了解這是否是「系統錯誤 53」訊息的原因，您可以使用 Portqry 查詢 TCP:445 端點。 如果篩選顯示 TCP:445 端點，則 TCP 連接埠會被封鎖。 查詢範例如下：

`g:\DataDump\Tools\Portqry>PortQry.exe -n [storage account name].file.core.windows.net -p TCP -e 445`

如果 TCP 445 是被網路路徑規則封鎖，您會看到下列輸出︰

**TCP port 445 (microsoft-ds service): FILTERED**

如需有關使用 Portqry 的詳細資訊，請參閱 [Portqry.exe 命令列公用程式的說明](https://support.microsoft.com/kb/310099)。

### <a name="solution-for-cause-2"></a>原因 2 的解決辦法
聯絡您的 IT 組織，要求開啟連接埠 445 輸出到 [Azure IP 範圍](https://www.microsoft.com/download/details.aspx?id=41653)。

### <a name="cause-3"></a>原因 3
如果用戶端上啟用 NTLMv1 通訊，也會收到「系統錯誤 53」。 啟用 NTLMv1 會使用戶端變得較不安全。 因此，Azure 檔案會封鎖通訊。 若要確認這是否為錯誤的原因，請確認下列登錄子機碼是設為 3︰

HKLM\SYSTEM\CurrentControlSet\Control\Lsa > LmCompatibilityLevel。

如需詳細資訊，請參閱 TechNet 上的 [LmCompatibilityLevel](https://technet.microsoft.com/library/cc960646.aspx) 主題。

### <a name="solution-for-cause-3"></a>原因 3 的解決辦法
若要解決此問題，將 HKLM\SYSTEM\CurrentControlSet\Control\Lsa 登錄機碼中 LmCompatibilityLevel 的值還原為預設值 3。

Azure 檔案僅支援 NTLMv2 驗證。 請確定群組原則有套用到用戶端。 這會防止發生此錯誤。 這也是公認最符合安全性的最佳作法。 如需詳細資訊，請參閱[如何利用群組原則將用戶端設定為使用 NTLMv2](https://technet.microsoft.com/library/jj852207\(v=ws.11\).aspx)。

建議的原則設定是**只傳送 NTLMv2 回應**。 這會對應到登錄值 3。 用戶端只會使用 NTLMv2 驗證，而且如果伺服器支援的話，用戶端會使用 NTLMv2 工作階段安全性。 網域控制站接受 LM、NTLM 和 NTLMv2 驗證。

<a id="netuse"></a>

## <a name="net-use-was-successful-but-dont-see-the-azure-file-share-mounted-in-windows-explorer"></a>net use 成功，但在 Windows 檔案總管中看不到掛接的 Azure 檔案共用
### <a name="cause"></a>原因
根據預設，Windows 檔案總管不會以系統管理員身分執行。 如果您從系統管理員命令提示字元執行 **net use**，會將網路磁碟機對應至「系統管理員身分」。 因為對應的磁碟機是以使用者為中心，如果磁碟機掛接在不同的使用者帳戶下，登入的使用者帳戶不會顯示此磁碟機。

### <a name="solution"></a>方案
從非系統管理員命令掛接共用。 或者，您可以依照[本 TechNet 主題](https://technet.microsoft.com/library/ee844140.aspx) 設定 **EnableLinkedConnections** 登錄值。

<a id="slashfails"></a>

## <a name="my-storage-account-contains--and-the-net-use-command-fails"></a>我的儲存體帳戶包含 "/"，net use 命令失敗
### <a name="cause"></a>原因
在命令提示字元 (cmd.exe) 中執行 **net use** 命令時，會被剖析為加上 "/" 作為命令列選項。 這會導致磁碟機對應失敗。

### <a name="solution"></a>方案
您可以使用下列其中一種方式來解決這個問題：

•    使用下列 PowerShell 命令：

`New-SmbMapping -LocalPath y: -RemotePath \\server\share  -UserName acountName -Password "password can contain / and \ etc"`

從批次檔可以這麼做

`Echo new-smbMapping ... | powershell -command –`

•    除非 "/" 是第一個字元，可以用雙引號括住索引鍵來解決此問題。 如果是，可使用互動模式分開輸入您的密碼，或者，重新產生索引鍵，以取得不是以正斜線 (/) 字元開頭的索引鍵。

<a id="accessfiledrive"></a>

## <a name="my-applicationservice-cannot-access-mounted-azure-files-drive"></a>我的應用程式/服務無法存取掛接的 Azure 檔案磁碟機
### <a name="cause"></a>原因
磁碟機是按每個使用者掛接。 如果您的應用程式或服務是在不同的使用者帳戶下執行，使用者就不會看到磁碟機。

### <a name="solution"></a>方案
從應用程所屬的相同使用者帳戶掛接磁碟機。 這可以使用 psexec 之類的工具。

或者，您可以建立具有和網路服務或系統帳戶相同權限的新使用者，然後在該帳戶下執行 **cmdkey** 和 **net use**。 使用者名稱應該是儲存體帳戶名稱，密碼應該是儲存體帳戶金鑰。 使用 **net use** 的另一個做法，是在 **net use** 命令的使用者名稱和密碼參數中傳遞儲存體帳戶名稱和金鑰。

遵循這些指示操作之後，您可能會收到下列錯誤訊息：「發生系統錯誤 1312。 指定的登入工作階段不存在。 可能已被終止。」，在您以系統/網路服務帳戶執行 **net use ** 時。 如果發生這種情況，請確定傳遞至 **net use** 的使用者名稱包含網域資訊 (例如，"[storage account name].file.core.windows.net")。

<a id="encryption"></a>

## <a name="error-you-are-copying-a-file-to-a-destination-that-does-not-support-encryption"></a>「您正將檔案複製到不支援加密的目的地」錯誤
### <a name="cause"></a>原因
Bitlocker 加密的檔案可以複製到 Azure 檔案。 不過，檔案儲存體不支援 NTFS EFS。 因此，在此情況下您可能會使用 EFS。 如果您透過 EFS 加密檔案，複製到檔案儲存體的複製作業可能會失敗，除非複製命令會解密複製的檔案。

### <a name="workaround"></a>因應措施
若要將檔案複製到檔案儲存體，您必須先將它解密。 您可以使用下列方法之一：

•    使用 **copy /d**。

•    設定下列登錄機碼︰

* Path=HKLM\Software\Policies\Microsoft\Windows\System
* Value type=DWORD
* Name = CopyFileAllowDecryptedRemoteDestination
* Value = 1

不過請注意，設定登錄機碼會影響所有複製到網路共用的複製作業。

<a id="errorhold"></a>

## <a name="host-is-down-error-on-existing-file-shares-or-the-shell-hangs-when-you-run-list-commands-on-the-mount-point"></a>在現有檔案共用上發生「主機當機」錯誤，或在掛接點上執行清單命令時殼層停止回應
### <a name="cause"></a>原因
當用戶端已經閒置一段時間，Linux 用戶端上會發生此錯誤。 發生此錯誤時，用戶端會中斷連接，然後用戶端連線逾時。

### <a name="solution"></a>方案
現在這個問題在 Linux 核心的[變更集](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/fs/cifs?id=4fcd1813e6404dd4420c7d12fb483f9320f0bf93)中已經修正，等待向下移植到 Linux 散發套件。

若要解決此問題，維持連線並避免進入閒置狀態，定期保存您寫入至 Azure 檔案共用中的檔案。 這必須是寫入作業，例如重寫檔案的建立/修改日期。 否則，您可能會收到快取的結果，而且您的作業可能不會觸發連線。

<a id="error15"></a>

## <a name="mount-error-115-when-you-try-to-mount-azure-files-on-the-linux-vm"></a>嘗試在 Linux VM 上掛接 Azure 檔案時發生「掛接錯誤 115」
### <a name="cause"></a>原因
Linux 散發套件尚未支援 SMB 3.0 中的加密功能。 在某些散發套件中，如果使用者因為功能遺失而嘗試使用 SMB 3.0 來掛接 Azure 檔案，可能會收到「115」錯誤訊息。

### <a name="solution"></a>方案
如果使用的 Linux SMB 用戶端不支援加密，在與檔案儲存體帳戶相同的資料中心上，從 Linux VM 使用 SMB 2.1 掛接 Azure 檔案。

<a id="delayproblem"></a>

## <a name="linux-vm-experiencing-random-delays-in-commands-like-ls"></a>使用類似 "ls" 的命令時 Linux VM 遇到隨機延遲
### <a name="cause"></a>原因
掛接命令中沒有 **serverino** 選項時可能會發生這個情況。 少了 **serverino**，ls 命令會對每個檔案執行 **stat**。

### <a name="solution"></a>方案
檢查 "/etc/fstab" 項目中的 **serverino**：

//azureuser.file.core.windows.net/wms/comer on /home/sampledir type cifs (rw,nodev,relatime,vers=2.1,sec=ntlmssp,cache=strict,username=xxx,domain=X, file_mode=0755,dir_mode=0755,serverino,rsize=65536,wsize=65536,actimeo=1)

如果沒有 **serverino** 選項，選取 **serverino** 選項即可先取消掛接再掛接 Azure 檔案。

## <a name="learn-more"></a>詳細資訊
* [在 Windows 上開始使用 Azure 檔案儲存體](storage-dotnet-how-to-use-files.md)
* [在 Linux 上開始使用 Azure 檔案儲存體](storage-how-to-use-files-linux.md)



<!--HONumber=Nov16_HO3-->


