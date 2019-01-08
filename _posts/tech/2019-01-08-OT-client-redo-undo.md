---
layout: post
title: 协同编辑OT算法client端UNDO和REDO的实现
category: 技术
tags: [协同编辑, OT]
keywords: 协同编辑,OT
---

客户端维护两个操作栈，undo操作栈和redo操作栈，数据结构如下：

    UndoManager {
        maxItems //undo，redo栈最大深度
        undoStack //undo操作栈
        redoStack //redo操作栈
    }

undo，redo处理会出现如下四种情况：

客户端有新的op操作：将 inverse(op) 压入undoStack
接收到服务端新的op操作：将服务端op操作应用到客户端文档，遍历undoStack所有操作进行OT转换，遍历redoStack所有操作进行OT转换
执行undo操作：将undoStack.pop应用到客户端文档，并发送给服务端，将inverse(undoStack.pop)压入redoStack
执行redo操作：将redoStack.pop应用到客户端文档，并发送给服务端，将inverse(redoStack.pop)压入undoStack

和单机文档编辑中的undo和redo相比，这里最大的区别是当服务端有新的操作发送到客户端时，客户端需要遍历undoStack和redoStack做相应的OT转换
