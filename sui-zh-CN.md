The Sui Smart Contracts Platform(Sui 智能合约平台)--中文白皮书
The MystenLabs Team
hello@mystenlabs.com

#1 INTRODUCTION(简介)
Sui is a decentralized permissionless smart contract platform biased towards low-latency management of assets. It uses the Move programming language to define assets as objects that may be owned by an address. Move programs define operations on these typed objects including custom rules for their creation, the transfer of these assets to new owners, and operations that mutate assets.
Sui 是一个去中心化的无许可智能合约平台，偏向于资产的低延迟管理。它使用 Move 编程语言将资产定义为地址可能拥有的对象。Move 程序定义了对这些类型化对象的操作，包括创建它们的自定义规则、将这些资产转移给新所有者以及改变资产的操作。

Sui is maintained by a permissionless set of authorities that play a role similar to validators or miners in other blockchain systems. It uses a Byzantine consistent broadcast protocol between authorities to ensure the safety of common operations on assets, ensuring lower latency and better scalability as compared to Byzantine agreement. It only relies on Byzantine agreement for the safety of shared objects. As well as governance operations and check-pointing, performed off the critical latency path. Execution of smart contracts is also naturally parallelized when possible. Sui supports light clients that can authenticate reads as well as full clients that may audit all transitions for integrity. These facilities allow for trust-minimized bridges to other blockchains.
Sui 由一组未经许可的权威机构维护，这些权威机构扮演着类似于其他区块链系统中的验证者或矿工的角色。它使用权威机构之间的拜占庭一致性广播协议来确保资产通用操作的安全性，与拜占庭协议相比确保更低的延迟和更好的可扩展性。它仅依赖拜占庭协议来确保共享对象的安全。以及治理操作和检查点，在关键延迟路径之外执行。在可能的情况下，智能合约的执行也会自然并行。 Sui 支持可以验证读取的轻客户端以及可以审计所有转换完整性的完整客户端。这些设施允许与其他区块链建立信任最小化的桥梁。

A native asset SUI is used to pay for gas for all operations. It is also used by its owners to delegate stake to authorities to operate Sui within epochs, and periodically, authorities are reconfigured according to the stake delegated to them. Used gas is distributed to authorities and their delegates according to their stake and their contribution to the operation of Sui.
原生资产 SUI 用于支付所有操作的 gas。它的所有者也使用它来将股份委托给当局以在世代内操作 Sui，并且定期地根据委托给他们的股份重新配置验证人。用过的gas根据他们的股份和他们对 Sui 运营的贡献分配给验证人和他们的代表。

This whitepaper is organized in two parts, with Sect. 2 describing the Sui programming model using the Move language, and Sect. 4 describing the operations of the permissionless decentralized system that ensures safety, liveness and performance for Sui.
本白皮书分为两部分，第2部分 使用 Move 语言描述 Sui 编程模型。 第4部分 描述了无需许可的去中心化系统的操作，以确保 Sui 的安全性、活跃性和性能。

#2 SUI SMART CONTRACT PROGRAMMING(SUI 智能合约编程)
Sui smart contracts are written in the Move[4] language. Move is safe and expressive, and its type system and data model naturally support the parallel agreement/execution strategies that make Sui scalable. Move is an open-source programming language for building smart contracts originally developed at Facebook for the Diem blockchain. The language is platform-agnostic, and in addition to being adopted by Sui, it has been gaining popularity on other platforms (e.g., 0L, StarCoin).
Sui 智能合约是用 Move[4] 语言编写的。 Move 安全且富有表现力，其类型系统和数据模型自然支持使 Sui 具有可扩展性的并行协议/执行策略。 Move 是一种开源编程语言，最初由 Facebook 为 Diem 区块链开发的智能合约。该语言与平台无关，除了被 Sui 采用外，它在其他平台（例如 0L、StarCoin）上也越来越受欢迎。

In this section we will discuss the main features of the Move language and explain how it is used to create and manage assets on Sui. A more thorough explanation of Move’s features can be found in the Move Programming Language book and more Sui-specific Move content can be found in the Sui Developer Portal , and a more formal description of Move in the context of Sui can be found in Section 3.
在本节中，我们将讨论 Move 语言的主要特性，并解释如何使用它在 Sui 上创建和管理资产。可以在 Move Programming Language 书中找到对 Move 功能的更详尽解释，可以在 Sui 开发人员门户中找到更多特定于 Sui 的 Move 内容，可以在第 3 节中找到 Sui 上下文中对 Move 的更正式描述.

##2.1 Overview(概述)
Sui’s global state includes a pool of programmable objects created and managed by Move packages that are collections of Move modules (see Section 2.1.1 for details) containing Move functions and types. Move packages themselves are also objects. Thus, Sui objects can be partitioned into two categories:
Sui 的全局状态包括由 Move 包创建和管理的可编程对象池，这些对象是包含 Move 函数和类型的 Move 模块的集合（详见第 2.1.1 节）。Move 包本身也是对象。因此，Sui 对象可以分为两类：

• Struct data values: Typed data governed by Move modules. Each object is a struct value with fields that can contain primitive types (e.g. integers, addresses), other objects, and non-object structs.
• 结构数据值：由Move模块管理的类型化数据。每个对象都是一个结构值，其字段可以包含基本类型（例如整数、地址）、其他对象和非对象结构。

• Package code values: a set of related Move bytecode modules published as an atomic unit. Each module in a package can depend both on other modules in that package and on modules in previously published packages.
• 包代码值：一组作为原子单元发布的相关Move字节码模块。包中的每个模块既可以依赖于该包中的其他模块，也可以依赖于先前发布的包中的模块。

Objects can encode assets (e.g., fungible or non-fungible tokens), capabilities granting the permission to call certain functions or create other objects, “smart contracts” that manage other assets, and so on–it’s up to the programmer to decide. The Move code to declare a custom Sui object type looks like this:
对象可以编码资产（例如，可替代或不可替代的代币）、授予调用某些函数或创建其他对象的权限的能力、管理其他资产的“智能合约”等等——这由程序员决定。声明自定义 Sui 对象类型的 Move 代码如下所示：

```go
struct Obj has key {
    id: VersionedID, // globally unique ID and version（全球唯一 ID 和版本）
    f: u64 // objects can have primitive fields（对象可以有原始字段）
    g: OtherObj // fields can also store other objects（字段还可以存储其他对象）
}
```

All structs representing Sui objects (but not all Move struct values) must have the id field and the key ability indicating that the value can be stored in Sui’s global object pool.
所有表示 Sui 对象的结构（但不是所有 Move 结构值）都必须具有 id 字段和指示该值可以存储在 Sui 的全局对象池中的键能力。

###2.1.1 Modules(模块)
A Move program is organized as a set of modules, each consisting of a list of struct declarations and function declarations. A module can import struct types from other modules and invoke functions declared by other modules.
Move 程序被组织为一组模块，每个模块由一系列结构声明和函数声明组成。一个模块可以从其他模块导入结构类型并调用其他模块声明的函数。

Values declared in one Move module can flow into another e.g., module OtherObj in the example above could be defined in a different module than the module defining Obj. This is different from most smart contract languages, which allow only unstructured bytes to flow across contract boundaries. However, Move is able to support this because it provides encapsulation features to help programmers write robustly safe code. Specifically, Move’s type system ensures that a type like Obj above can only be created, destroyed, copied, read, and written by functions inside the module that declares the type. This allows a module to enforce strong invariants on its declared types that continue to hold even when they flow across smart contract trust boundaries.
在一个 Move 模块中声明的值可以流入另一个模块，例如，上面示例中的模块 OtherObj 可以在与定义 Obj 的模块不同的模块中定义。这与大多数智能合约语言不同，后者只允许非结构化字节流过合约边界。但是，Move 能够支持这一点，因为它提供了封装功能来帮助程序员编写稳健安全的代码。具体来说，Move 的类型系统确保像上面的 Obj 这样的类型只能由声明该类型的模块内部的函数创建、销毁、复制、读取和写入。这允许模块对其声明的类型强制执行强不变性，即使它们流过智能合约信任边界，这些类型也会继续保持不变。

###2.1.2 Transactions and Entrypoints(交易和入口点)
The global object pool is updated via transactions that can create, destroy, read, and write objects. A transaction must take each existing object it wishes to operate on as an input. In addition, a transaction must include the versioned ID of a package object, the name of a module and function inside that package, and arguments to the function (including input objects). For example, to call the function
全局对象池通过可以创建、销毁、读取和写入对象的交易进行更新。事务必须将它希望操作的每个现有对象作为输入。此外，一个交易必须包含包对象的版本 ID、包内模块和函数的名称以及函数的参数（包括输入对象）。例如调用函数

```
public fun entrypoint(
o1: Obj, o2: &mut Obj, o3: &Obj, x: u64, ctx: &mut TxContext
) { ... }
```

a transaction must supply ID’s for three distinct objects whose type is Obj and an integer to bind to x. The TxContext is a special parameter filled in by the runtime that contains the sender address and information required to create new objects.
交易必须为类型为 Obj 的三个不同对象提供 ID，并提供一个绑定到 x 的整数。 TxContext 是运行时填充的特殊参数，包含发送者地址和创建新对象所需的信息。

Inputs to an entrypoint (and more generally, to any Move function) can be passed with different mutability permissions encoded in the type. An Obj input can be read, written, transferred, or destroyed. A &mut Obj input can only be read or written, and a &Obj can only be read. The transaction sender must be authorized to use each of the input objects with the specified mutability permissions– see Section 4.4 for more detail.
可以使用在类型中编码的不同可变性权限来传递对入口点（更一般地说，对任何 Move 函数）的输入。可以读取、写入、传输或销毁 Obj 输入。 &mut Obj 输入只能读或写，&Obj 只能读。交易发送者必须被授权使用每个具有指定可变性权限的输入对象——更多细节请参见第 4.4 节。

###2.1.3 Creating and Transferring Objects(创建和传输对象) 
Programmers can create objects by using the TxContext passed into the entrypoint to generate a fresh ID for the object:
程序员可以通过使用传递到入口点的 TxContext 为对象生成新的 ID 来创建对象：

```
public fun create_then_transfer(
f: u64, g: OtherObj, o1: Obj, ctx: &mut TxContext
) {
    let o2 = Obj { id: TxContext::fresh_id(ctx), f, g };
    Transfer::transfer(o1, TxContext:sender());
    Transfer::transfer(o2, TxContext:sender());
}
```

This code takes two objects of type OtherObj and Obj as input, uses the first one and the generated ID to create a new Obj, and then transfers both Obj objects to the transaction sender. Once an object has been transferred, it flows into the global object pool and cannot be accessed by code in the remainder of the transaction. The Transfer module is part of the Sui standard library, which includes functions for transferring objects to user addresses and to other objects.
此代码将两个类型为 OtherObj 和 Obj 的对象作为输入，使用第一个和生成的 ID 创建一个新的 Obj，然后将这两个 Obj 对象传输给交易发送方。一旦一个对象被转移，它就会流入全局对象池，并且在交易的其余部分不能被代码访问。 Transfer 模块是 Sui 标准库的一部分，其中包括将对象传输到用户地址和其他对象的函数。

We note that if the programmer code neglected to include one of the transfer calls, this code would be rejected by the Move type system. Move enforces resource safety protections to ensure that objects cannot be created without permission, copied, or accidentally destroyed. Another example of resource safety would be an attempt to transfer the same object twice, which would also be rejected by the Move type system.
我们注意到，如果程序员代码忽略了其中一个传输调用，则该代码将被 Move 类型系统拒绝。 Move 强制执行资源安全保护，以确保对象不会在未经许可的情况下创建、复制或意外销毁。资源安全的另一个例子是尝试传输同一个对象两次，这也会被 Move 类型系统拒绝。

#3 THE SUI PROGRAMMING MODEL(SUI 编程模型)

In this section, we expand on the informal description of the Sui programming model from Section 2 by presenting detailed semantic definitions. The previous section showed examples of Move source code; here we define the structure of Move bytecode. Developers write, test, and formally verify Move source code locally, then compile it to Move bytecode before publishing it to the blockchain. Any Move bytecode be published on-chain must pass through a bytecode verifier to ensure that it satisfies key properties such as type, memory, and resource safety.
在本节中，我们通过提供详细的语义定义来扩展第 2 节中对 Sui 编程模型的非正式描述。上一节展示了 Move 源代码的示例；这里我们定义了 Move 字节码的结构。开发人员在本地编写、测试和正式验证 Move 源代码，然后在将其发布到区块链之前将其编译为 Move 字节码。任何在链上发布的 Move 字节码都必须通过字节码验证器，以确保其满足类型、内存和资源安全等关键属性。

As mentioned in Section 2, Move is a platform-agnostic language which can be adapted to fit specific needs of different systems without forking the core language. In the following description, we define both concepts from core Move language (denoted in black text) and Sui-specific features extending the core Move language(denoted with orange text).
如第 2 节所述，Move 是一种与平台无关的语言，无需分叉核心语言即可适应不同系统的特定需求。在下面的描述中，我们定义了核心 Move 语言的概念（用黑色文本表示）和扩展核心 Move 语言的特定于 Sui 的功能（用橙色文本表示）。

Module       = ModuleName× (StructName ⇀ StructDecl)×(FunName ⇀ FunDecl) × <font color=#FFA500>FunDecl</font>
GenericParam = [Ability]
StructDecl   = (FieldName ⇀ StorableType)×[Ability] × [GenericParam]
FunDecl      = [Type] [Type] × [Instr] × [GenericParam]
Instr        = <font color=#FFA500>TransferToAddr | TransferToObj | ShareMut | ShareImmut | ... </font>

Table 1: Module

Move code is organized into modules whose structure is defined in Table 1. A module consists of a collection of named struct declarations and a collection of named function declarations (examples of these declaration are provided in Section 2.1). A module also contains a special function declaration serving as the module initializer. This function is invoked exactly once at the time the module is published on-chain.
Move 代码被组织成模块，其结构在表 1 中定义。模块由一组命名结构声明和一组命名函数声明组成（这些声明的示例在第 2.1 节中提供）。模块还包含用作模块初始值设定项的特殊函数声明。该函数在模块发布到链上时被调用一次。

A struct declaration is a collection of named fields, where a field name is mapped to a storeable type. Its declaration also includes an optional list of abilities (see Section 2 for a description of storeable types and abilities). A struct declaration may also include a list of generic parameters with ability constraints, in which case we call it a generic struct declaration, for example struct Wrapper<T: copy>{ t: T }. A generic parameter represents a type to be used when declaring struct fields – it is unknown at the time of struct declaration, with a concrete type provided when the struct is instantiated(i.e., as struct value is created).
结构声明是命名字段的集合，其中字段名称映射到可存储类型。它的声明还包括一个可选的能力列表（有关可存储类型和能力的描述，请参阅第 2 节）。结构声明还可以包括具有能力约束的泛型参数列表，在这种情况下我们称其为泛型结构声明，例如 struct Wrapper<T: copy>{ t: T }。泛型参数表示在声明结构字段时要使用的类型——在结构声明时它是未知的，具体类型在结构被实例化时提供（即，作为结构值被创建）。

A function declaration includes a list of parameter types, a list of return types, and a list of instructions forming the function’s body.
函数声明包括参数类型列表、返回类型列表和构成函数主体的指令列表。
A function declaration may also include a list of generic parameters with ability constraints, in which case we call it a generic function declaration, for example fun unwrap<T: copy>(p: Wrapper<T>){}.
一个函数声明也可能包括一个带有能力约束的泛型参数列表，在这种情况下我们称它为泛型函数声明，例如 fun unwrap<T: copy>(p: Wrapper<T>){}。
Similarly to struct declarations, a generic parameter represents a type unknown at function declaration time, but which is nevertheless used when declaring function parameters, return values and a function body (concrete type is provided when a function is called).
与结构声明类似，泛型参数表示函数声明时未知的类型，但在声明函数参数、返回值和函数体时仍会使用它（调用函数时提供具体类型）。

Instructions that can appear in a function body include all ordinary Move instructions with the exception of global storage instructions (e.g., move_to, move_from, borrow_global).
可以出现在函数体中的指令包括除全局存储指令（例如 move_to、move_from、borrow_global）之外的所有普通 Move 指令。
See [14] for a complete list of core Move’s instructions and their semantics. In Sui persistent storage is supported via Sui’s global object pool rather than the account-based global storage of core Move.
有关核心 Move 指令及其语义的完整列表，请参见 [14]。在 Sui 中，持久存储是通过 Sui 的全局对象池而不是核心 Move 的基于帐户的全局存储来支持的。

There are four Sui-specific object operations. Each of these operations changes the ownership metadata of the object (see Section 3.3) and returns it to the global object pool. Most simply, a Sui object can be transferred to the address of a Sui end-user.
有四个特定于 Sui 的对象操作。这些操作中的每一个都会更改对象的所有权元数据（请参阅第 3.3 节）并将其返回到全局对象池。最简单的是，可以将 Sui 对象传输到 Sui 最终用户的地址。
An object can also be transferred to another parent object–this operation requires the caller to supply a mutable reference to the parent object in addition to the child object.
一个对象也可以转移到另一个父对象——这个操作需要调用者除了子对象之外还提供一个对父对象的可变引用。
An object can be mutably shared so it can be read/written by anyone in the Sui system. Finally, an object can be immutably shared so it can be read by anyone in the Sui system, but not written by anyone.
一个对象可以可变共享，因此它可以被 Sui 系统中的任何人读/写。最后，一个对象可以被不可变地共享，所以它可以被 Sui 系统中的任何人读取，但不能被任何人写入。

The ability to distinguish between different kinds of ownership is a unique feature of Sui. In other blockchain platforms we are aware of, every contract and object is mutably shared. As we will explain in Section 4, Sui leverages this information for parallel transaction execution (for all transactions) and parallel agreement(for transactions involving objects without shared mutability).
区分不同所有制的能力是Sui的一个独特之处。在我们知道的其他区块链平台中，每个合约和对象都是可变共享的。正如我们将在第 4 节中解释的那样，Sui 利用此信息进行并行事务执行（对于所有事务）和并行协议（对于涉及没有共享可变性的对象的交易）。

##3.2 Types and Abilities(类型和能力)

PrimType       = {address, id, bool, u8, u64, . . .}
StructType     = ModuleName × StructName× [StorableType]
StorableType   = PrimType ⊎ StructType ⊎ GenericType ⊎ VectorType
VectorType     = StorableType
GenericType    = N
MutabilityQual = {mut, immut}
ReferenceType  = StorableType × MutabilityQual
Type           = ReferenceType ⊎ StorableType
Ability        = {key, store, copy, drop}

Table 2: Types and Abilities

A Move program manipulates both data stored in Sui global object pool and transient data created when the Move program executes.
Move 程序操作存储在 Sui 全局对象池中的数据和 Move 程序执行时创建的瞬态数据。
Both objects and transient data are Move values at the language level.
对象和瞬态数据都是语言级别的 Move 值。
However, not all values are created equal – they may have different properties and different structure as prescribed by their types.
然而，并非所有的值都是平等的——它们可能具有不同的属性和不同的结构，正如它们的类型所规定的那样。

The types used in Move are defined in Table 2. Move supports many of the same primitive types supported in other programming languages, such as a boolean type or unsigned integer types of various sizes. In addition, core Move has an address type representing an end-user in the system that is also used to identify the sender of a transaction and (in Sui) the owner of an object. Finally, Sui defines an id type representing an identity of a Sui object– see Section 3.3 for details.
Move 中使用的类型在表 2 中定义。Move 支持其他编程语言支持的许多相同的原始类型，例如布尔类型或各种大小的无符号整数类型。此外，核心 Move 具有代表系统中最终用户的地址类型，该地址类型也用于识别交易的发送者和（在 Sui 中）对象的所有者。最后，Sui 定义了一个 id 类型，表示 Sui 对象的身份——详情请参阅第 3.3 节。

A struct type describes an instance (i.e., a value) of a struct declared in a given module (see Section 3.1 for information on struct declarations).
结构类型描述了在给定模块中声明的结构的实例（即值）（有关结构声明的信息，请参阅第 3.1 节）。
A struct type representing a generic struct declaration (i.e., generic struct type) includes a list of storeable types – this list is the counterpart of the generic parameter list in the struct declaration. A storeable type can be either a concrete type (a primitive or a struct) or a generic type. We call such types storeable because they can appear as fields of structs and in objects stored persistently on-chain, whereas reference types cannot.
表示通用结构声明（即通用结构类型）的结构类型包括一个可存储类型列表——该列表是结构声明中通用参数列表的对应部分。可存储类型可以是具体类型（原始类型或结构）或泛型类型。我们称这种类型为可存储的，因为它们可以作为结构的字段出现，也可以出现在链上持久存储的对象中，而引用类型则不能。

For example, the Wrapper<u64> struct type is a generic struct type parameterized with a concrete (primitive) storeable type u64 – this kind of type can be used to create a struct instance (i.e.,value).
例如，Wrapper<u64> 结构类型是用具体（原始）可存储类型 u64 参数化的通用结构类型——这种类型可用于创建结构实例（即值）。
On the other hand, the same generic struct type can be parameterized with a generic type (e.g., struct Parent<T> { w: Wrapper<T> }) coming from a generic parameter of the enclosing struct or function declaration – this kind of type can be used to declare struct fields, function params, etc. Structurally, a generic type is an integer index(defined as N in Table 5) into the list of generic parameters in the enclosing struct or function declaration.
另一方面，相同的泛型结构类型可以用来自封闭结构或函数声明的泛型参数的泛型类型（例如，struct Parent<T> { w: Wrapper<T> }）进行参数化——这种type 可用于声明结构字段、函数参数等。在结构上，泛型类型是一个整数索引（在表 5 中定义为 N）到封闭结构或函数声明中的泛型参数列表中。

A vector type in Move describes a variable length collection of homogenous values. A Move vector can only contain storeable types, and it is also a storeable type itself.
Move 中的向量类型描述了同质值的可变长度集合。 Move 向量只能包含可存储类型，并且它本身也是可存储类型。

A Move program can operate directly on values or access them indirectly via references. A reference type includes both the storeable type referenced and a mutability qualifier used to determine (and enforce) whether a value of a given type can be read and written(mut) or only read (immut). Consequently, the most general form of a Move value type (Type in Table 2) can be either a storeable type or a reference type.
Move 程序可以直接对值进行操作或通过引用间接访问它们。引用类型包括所引用的可存储类型和用于确定（和强制执行）给定类型的值是可读可写 (mut) 还是仅可读 (immut) 的可变性限定符。因此，Move 值类型（表 2 中的类型）的最一般形式可以是可存储类型或引用类型。

Finally, abilities in Move control what actions are permissible for values of a given type, such as whether a value of a given type can be copied (duplicated). Abilities constraint struct declarations and generic type parameters. The Move bytecode verifier is responsible for ensuring that sensitive operations like copies can only be performed on types with the corresponding ability.
最后，Move 中的能力控制对给定类型的值允许的操作，例如是否可以复制（复制）给定类型的值。能力约束结构声明和泛型类型参数。 Move 字节码验证器负责确保像复制这样的敏感操作只能在具有相应能力的类型上执行。
