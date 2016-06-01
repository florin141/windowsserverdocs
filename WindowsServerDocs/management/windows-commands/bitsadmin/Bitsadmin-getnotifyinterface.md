---
title: Bitsadmin getnotifyinterface
ms.custom: na
ms.prod: windows-server-2012
ms.reviewer: na
ms.suite: na
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 40bf9dd8-b167-406a-80a6-a5a6f1b8cf7f
---
# Bitsadmin getnotifyinterface
Determines if another program has registered a COM callback interface for the specified job.

## Syntax

```
bitsadmin /GetNotifyInterface <Job>
```

## Parameters

|Parameter|Description|
|-------------|---------------|
|Job|The job's display name or GUID|

## Remarks
Displays REGISTERED or UNREGISTERED.

> [!NOTE]
> It is not possible to determine the program that registered the callback interface.

## <a name="BKMK_examples"></a>Examples
The following example retrieves the notify interface for the job named *myDownloadJob*.

```
C:\>bitsadmin /GetNotifyInterface myDownloadJob
```

## Additional references
[Command-Line Syntax Key](Command-Line-Syntax-Key.md)

