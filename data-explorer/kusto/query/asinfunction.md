---
title: asin() - Azure Data Explorer
description: This article describes asin() in Azure Data Explorer.
ms.reviewer: alexans
ms.topic: reference
ms.date: 10/23/2018
---
# asin()

Returns the angle whose sine is the specified number (the inverse operation of [`sin()`](sinfunction.md)).

## Syntax

`asin(`*x*`)`

## Arguments

* *x*: A real number in range [-1, 1].

## Returns

* The value of the arc sine of `x`
* `null` if `x` < -1 or `x` > 1
