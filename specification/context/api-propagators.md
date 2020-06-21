<!--
# Propagators API
-->

# Propagator API

<!--
<details>
<summary>
Table of Contents
</summary>
-->

<details>
<summary>
目次
</summary>

<!--
- [Overview](#overview)
- [HTTP Text Format](#http-text-format)
  - [Fields](#fields)
  - [Inject](#inject)
    - [Setter argument](#setter)
      - [Set](#set)
  - [Extract](#extract)
    - [Getter argument](#getter)
      - [Get](#get)
- [Composite Propagator](#composite-propagator)
- [Global Propagators](#global-propagators)
-->

- [概要](#概要)
- [HTTP テキストフォーマット](#HTTP-テキストフォーマット)
  - [フィールド](#フィールド)
  - [注入(inject)](#注入inject)
    - [Setter argument](#setter)
      - [Set](#set)
  - [Extract](#extract)
    - [Getter argument](#getter)
      - [Get](#get)
- [Composite Propagator](#composite-propagator)
- [Global Propagators](#global-propagators)

<!--
</details>
-->

</details>

<!--
## Overview
-->

## 概要

<!--
Cross-cutting concerns send their state to the next process using
`Propagator`s, which are defined as objects used to read and write
context data to and from messages exchanged by the applications.
Each concern creates a set of `Propagator`s for every supported `Format`.
-->

関連するコンポーネントは自分自身の状態を `Propagator` を使って次のプロセスに送信します。`Propagator`はアプリケーションが交換するメッセージとの間でコンテキストデータを読み書きするためのオブジェクトとして定義されています。各関連するコンポーネントは、サポートされている `Format`全部に対して `Propagator` のセットを作成します。

<!--
Propagators leverage the `Context` to inject and extract data for each
cross-cutting concern, such as traces and correlation context.
-->

Propagatorは `Context` を利用して、関連するコンポーネントそれぞれに対してTraceやCorrelationContextなどのデータを注入したり抽出したりします。

<!--
The Propagators API is expected to be leveraged by users writing
instrumentation libraries.
-->

Propagator APIは計装ライブラリを書いているユーザーが活用することが期待されています。

<!--
The Propagators API currently consists of one `Format`:
-->

Propagator APIは現在1つの `Format` から構成されています。

<!--
- `HTTPTextFormat` is used to inject values into and extract values from carriers as text that travel
  in-band across process boundaries.
-->

- `HTTPTextFormat` はプロセスの境界を越えてインバンドで移動するキャリアに、関連するコンポーネントの値をテキストとして注入・抽出することに使われます。
<!--
A binary `Format` will be added in the future.
-->

将来的にはバイナリの `Format` が追加される予定です。

<!--
## HTTP Text Format
-->

## HTTP テキストフォーマット

<!--
`HTTPTextFormat` is a formatter that injects and extracts a cross-cutting concern
value as text into carriers that travel in-band across process boundaries.
-->

`HTTP テキストフォーマット` は、プロセスの境界を越えてインバンドで移動するキャリアに、関連するコンポーネントの値をテキストとして注入・抽出するフォーマッタです。

<!--
Encoding is expected to conform to the HTTP Header Field semantics. Values are often encoded as
RPC/HTTP request headers.
-->

エンコードは HTTP ヘッダーフィールドのセマンティクスに準拠することが期待されます。値はしばしばRPC/HTTPリクエストヘッダとしてエンコードされます。

<!--
The carrier of propagated data on both the client (injector) and server (extractor) side is
usually an http request. Propagation is usually implemented via library-specific request
interceptors, where the client-side injects values and the server-side extracts them.
-->

クライアント側(注入側)とサーバ側(抽出側)の両方で伝搬されるデータのキャリアは、通常はHTTPリクエストです。伝搬は通常、ライブラリ固有のリクエストインターセプターを介して実装され、 クライアント側が値を注入し、サーバ側が値を抽出します。


<!--
`HTTPTextFormat` MUST expose the APIs that injects values into carriers,
and extracts values from carriers.
-->

HTTPTextFormat` はキャリアに値を注入し、キャリアから値を抽出するAPIを公開する必要があります(MUST)。

<!--
### Fields
-->

### フィールド

<!--
The propagation fields defined. If your carrier is reused, you should delete the fields here
before calling [inject](#inject).
-->

定義されている伝搬フィールドです(???)。キャリアを再利用する場合は、[注入(inject)](#注入inject)を呼び出す前にこのフィールドを削除する必要があります。

<!--
For example, if the carrier is a single-use or immutable request object, you don't need to
clear fields as they couldn't have been set before. If it is a mutable, retryable object,
successive calls should clear these fields first.
-->

例えば、キャリアが一回のみ使われる、または不変のリクエストオブジェクトであれば、フィールドを削除する必要はありません。もし、変更可能で何度も使われる可能性があるオブジェクトであれば、連続した呼び出しは最初にこれらのフィールドをクリアしなければなりません。

<!--
The use cases of this are:
-->

ユースケースを以下に示します。

<!--
- allow pre-allocation of fields, especially in systems like gRPC Metadata
- allow a single-pass over an iterator
-->

- 特に gRPC のメタデータのようなシステムでフィールドの事前割り当てを可能にする
- イテレータ上で一回通しで実行するだけを可能にする (???single-passとは一回でパスするってこと？)

<!--
Returns list of fields that will be used by this formatter.
-->

このフォーマッタが使用するフィールドのリストを返します。(??? この文章唐突なんだけど)

<!--
### Inject
-->

### 注入(inject)

<!--
Injects the value downstream. For example, as http headers.
-->

例えばhttpヘッダとして、次に渡すプロセスのために値を注入します。

<!--
Required arguments:
-->

必要な引数:

<!--
- A `Context`. The Propagator MUST retrieve the appropriate value from the `Context` first, such as `SpanContext`, `CorrelationContext` or another cross-cutting concern context. For languages supporting current `Context` state, this argument is OPTIONAL, defaulting to the current `Context` instance.
- the carrier that holds propagation fields. For example, an outgoing message or http request.
- the `Setter` invoked for each propagation key to add or remove.
-->

- `Context`: Propagator は最初に `SpanContext` や `CorrelationContext` あるいは他の関連するコンポーネントの `Context` から適切な値を取得する必要があります(MUST)。現在の `Context` をサポートする言語では、この引数はオプションであり、デフォルトは現在の `Context` インスタンスになります。
- 伝搬用のフィールドを保持するキャリア: 例えば、発信メッセージやhttpリクエストなどです。
- 追加または削除する伝搬キーごとに呼び出される `Setter`

<!--
#### Setter argument
-->

#### Setter引数

<!--
Setter is an argument in `Inject` that sets value into given field.
-->

Setterは指定されたフィールドに値をセットする `Inject` のための引数です。


<!--
`Setter` allows a `HTTPTextFormat` to set propagated fields into a carrier.
-->

`Setter` は 伝搬されたフィールドをキャリアに`HTTPTextFormat` を設定します。

<!--
`Setter` MUST be stateless and allowed to be saved as a constant to avoid runtime allocations. One of the ways to implement it is `Setter` class with `Set` method as described below.
-->

`Setter` はステートレスでなければならず、実行時の割り当てを避けるために定数として保存できる必要があります(MUST)。これを実装する方法の一つは、以下のように `Set` メソッドを持つ `Setter` クラスです。

<!--
##### Set
-->

##### Set

<!--
Replaces a propagated field with the given value.
-->

伝搬されたフィールドを指定された値に置き換えます。

<!--
Required arguments:
-->

必要な引数:

<!--
- the carrier holds propagation fields. For example, an outgoing message or http request.
- the key of the field.
- the value of the field.
-->

- the carrier holds propagation fields. For example, an outgoing message or http request.
- the key of the field.
- the value of the field.

<!--
The implemenation SHOULD preserve casing (e.g. it should not transform `Content-Type` to `content-type`) if the used protocol is case insensitive, otherwise it MUST preserve casing.
-->

The implemenation SHOULD preserve casing (e.g. it should not transform `Content-Type` to `content-type`) if the used protocol is case insensitive, otherwise it MUST preserve casing.

<!--
### Extract
-->

### Extract

<!--
Extracts the value from an incoming request. For example, from the headers of an HTTP request.
-->

Extracts the value from an incoming request. For example, from the headers of an HTTP request.

<!--
If a value can not be parsed from the carrier for a cross-cutting concern,
the implementation MUST NOT throw an exception. It MUST store a value in the `Context`
that the implementation can recognize as a null or empty value.
-->

If a value can not be parsed from the carrier for a cross-cutting concern, the implementation MUST NOT throw an exception. It MUST store a value in the `Context` that the implementation can recognize as a null or empty value.

<!--
Required arguments:
-->

Required arguments:

<!--
- A `Context`. For languages supporting current `Context` state this argument is OPTIONAL, defaulting to the current `Context` instance.
- the carrier holds propagation fields. For example, an outgoing message or http request.
- the instance of `Getter` invoked for each propagation key to get.
-->

- A `Context`. For languages supporting current `Context` state this argument is OPTIONAL, defaulting to the current `Context` instance. - the carrier holds propagation fields. For example, an outgoing message or http request. - the instance of `Getter` invoked for each propagation key to get.

<!--
Returns a new `Context` derived from the `Context` passed as argument,
containing the extracted value, which can be a `SpanContext`,
`CorrelationContext` or another cross-cutting concern context.
-->

Returns a new `Context` derived from the `Context` passed as argument, containing the extracted value, which can be a `SpanContext`, `CorrelationContext` or another cross-cutting concern context.

<!--
If the extracted value is a `SpanContext`, its `IsRemote` property MUST be set to true.
-->

If the extracted value is a `SpanContext`, its `IsRemote` property MUST be set to true.

<!--
#### Getter argument
-->

#### Getter argument

<!--
Getter is an argument in `Extract` that get value from given field
-->

Getter is an argument in `Extract` that get value from given field

<!--
`Getter` allows a `HttpTextFormat` to read propagated fields from a carrier.
-->

`Getter` allows a `HttpTextFormat` to read propagated fields from a carrier.

<!--
`Getter` MUST be stateless and allowed to be saved as a constant to avoid runtime allocations. One of the ways to implement it is `Getter` class with `Get` method as described below.
-->

`Getter` MUST be stateless and allowed to be saved as a constant to avoid runtime allocations. One of the ways to implement it is `Getter` class with `Get` method as described below.

<!--
##### Get
-->

##### Get

<!--
The Get function MUST return the first value of the given propagation key or return null if the key doesn't exist.
-->

The Get function MUST return the first value of the given propagation key or return null if the key doesn't exist.

<!--
Required arguments:
-->

Required arguments:

<!--
- the carrier of propagation fields, such as an HTTP request.
- the key of the field.
-->

- the carrier of propagation fields, such as an HTTP request. - the key of the field.

<!--
The Get function is responsible for handling case sensitivity. If the getter is intended to work with a HTTP request object, the getter MUST be case insensitive. To improve compatibility with other text-based protocols, text `Format` implementions MUST ensure to always use the canonical casing for their attributes. NOTE: Cannonical casing for HTTP headers is usually title case (e.g. `Content-Type` instead of `content-type`).
-->

The Get function is responsible for handling case sensitivity. If the getter is intended to work with a HTTP request object, the getter MUST be case insensitive. To improve compatibility with other text-based protocols, text `Format` implementions MUST ensure to always use the canonical casing for their attributes. NOTE: Cannonical casing for HTTP headers is usually title case (e.g. `Content-Type` instead of `content-type`).

<!--
## Injectors and Extractors as Separate Interfaces
-->

## Injectors and Extractors as Separate Interfaces

<!--
Languages can choose to implement a `Propagator` for a format as a single object
exposing `Inject` and `Extract` methods, or they can opt to divide the
responsibilities further into individual `Injector`s and `Extractor`s. A
`Propagator` can be implemented by composing individual `Injector`s and
`Extractors`.
-->

Languages can choose to implement a `Propagator` for a format as a single object exposing `Inject` and `Extract` methods, or they can opt to divide the responsibilities further into individual `Injector`s and `Extractor`s. A `Propagator` can be implemented by composing individual `Injector`s and `Extractors`.

<!--
## Composite Propagator
-->

## Composite Propagator

<!--
Implementations MUST offer a facility to group multiple `Propagator`s
from different cross-cutting concerns in order to leverage them as a
single entity.
-->

Implementations MUST offer a facility to group multiple `Propagator`s from different cross-cutting concerns in order to leverage them as a single entity.

<!--
A composite propagator can be built from a list of propagators, or a list of
injectors and extractors. The resulting composite `Propagator` will invoke the `Propagator`s, `Injector`s, or `Extractor`s, in the order they were specified.
-->

A composite propagator can be built from a list of propagators, or a list of injectors and extractors. The resulting composite `Propagator` will invoke the `Propagator`s, `Injector`s, or `Extractor`s, in the order they were specified.

<!--
Each composite `Propagator` will be bound to a specific `Format`, such
as `HttpTextFormat`, as different `Format`s will likely operate on different
data types.
There MUST be functions to accomplish the following operations.
-->

Each composite `Propagator` will be bound to a specific `Format`, such as `HttpTextFormat`, as different `Format`s will likely operate on different data types. There MUST be functions to accomplish the following operations.

<!--
- Create a composite propagator
- Extract from a composite propagator
- Inject into a composite propagator
-->

- Create a composite propagator - Extract from a composite propagator - Inject into a composite propagator

<!--
### Create a Composite Propagator
-->

### Create a Composite Propagator

<!--
Required arguments:
-->

Required arguments:

<!--
- A list of `Propagator`s or a list of `Injector`s and `Extractor`s.
-->

- A list of `Propagator`s or a list of `Injector`s and `Extractor`s.

<!--
Returns a new composite `Propagator` with the specified `Propagator`s.
-->

Returns a new composite `Propagator` with the specified `Propagator`s.

<!--
### Extract
-->

### Extract

<!--
Required arguments:
-->

Required arguments:

<!--
- A `Context`.
- The carrier that holds propagation fields.
- The instance of `Getter` invoked for each propagation key to get.
-->

- A `Context`. - The carrier that holds propagation fields. - The instance of `Getter` invoked for each propagation key to get.

<!--
### Inject
-->

### Inject

<!--
Required arguments:
-->

Required arguments:

<!--
- A `Context`.
- The carrier that holds propagation fields.
- The `Setter` invoked for each propagation key to add or remove.
-->

- A `Context`. - The carrier that holds propagation fields. - The `Setter` invoked for each propagation key to add or remove.

<!--
## Global Propagators
-->

## Global Propagators

<!--
Implementations MAY provide global `Propagator`s for
each supported `Format`.
-->

Implementations MAY provide global `Propagator`s for each supported `Format`.

<!--
If offered, the global `Propagator`s should default to a composite `Propagator`
containing W3C Trace Context and Correlation Context `Propagator`s,
in order to provide propagation even in the presence of no-op
OpenTelemetry implementations.
-->

If offered, the global `Propagator`s should default to a composite `Propagator` containing W3C Trace Context and Correlation Context `Propagator`s, in order to provide propagation even in the presence of no-op OpenTelemetry implementations.

<!--
### Get Global Propagator
-->

### Get Global Propagator

<!--
This method MUST exist for each supported `Format`.
-->

This method MUST exist for each supported `Format`.

<!--
Returns a global `Propagator`. This usually will be composite instance.
-->

Returns a global `Propagator`. This usually will be composite instance.

<!--
### Set Global Propagator
-->

### Set Global Propagator

<!--
This method MUST exist for each supported `Format`.
-->

This method MUST exist for each supported `Format`.

<!--
Sets the global `Propagator` instance.
-->

Sets the global `Propagator` instance.

<!--
Required parameters:
-->

Required parameters:

<!--
- A `Propagator`. This usually will be a composite instance.
-->

- A `Propagator`. This usually will be a composite instance.

