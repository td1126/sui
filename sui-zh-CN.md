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

##3.3 Objects and Ownership(对象和所有权)

TxDigest    = 𝐶𝑜𝑚(Tx)
ObjID       = 𝐶𝑜𝑚(TxDigest × N)
SingleOwner = Addr ⊎ ObjID
Shared      = {shared_mut, shared_immut}
Ownership   = SingleOwner ⊎ Shared
StructObj   = StructType × Struct
ObjContents = StructObj ⊎ Package
Obj         = ObjContents × ObjID × Ownership × Version

Table 3: Objects and Ownership

Each Sui object has a globally unique identifier (ObjID in Table 3)that serves as the persistent identity of the object as it flows between owners and into and out of other objects.
每个Sui对象都有一个全局唯一的标识符（表3中的ObjID），当对象在所有者之间流动以及进出其他对象时，该标识符充当对象的持久标识。
This ID is assigned to the object by the transaction that creates it. An object ID is created by applying a collision-resistant hash function to the contents of the current transaction and to a counter recording how many objects the transaction has created.
此ID由创建对象的交易分配给对象。对象ID是通过将抗冲突哈希函数应用于当前交易的内容和记录交易创建了多少对象的计数器来创建的。
A transaction (and thus its digest) is guaranteed to be unique due to constraints on the input objects of the transaction, as we will explain subsequently.
由于对交易的输入对象的约束，交易（以及它的摘要）被保证是唯一的，正如我们随后将解释的那样。

In addition to an ID, each object carries metadata about its ownership. An object is either uniquely owned by an address or another object, shared with write/read permissions, or shared with only read permissions.
除了ID之外，每个对象还携带有关其所有权的元数据。一个对象要么由一个地址或另一个对象唯一拥有，要么与写/读权限共享，要么仅与读权限共享。
The ownership of an object determines whether and how a transaction can use it as an input. Broadly, a uniquely owned object can only be used in a transaction initiated by its owner or including its parent object as an input, whereas a shared object can be used by any transaction, but only with the specified mutability permissions.
对象的所有权决定了交易是否可以将其用作输入以及如何将其用作输入。从广义上讲，唯一拥有的对象只能在其所有者发起的交易中使用，或者包括其父对象作为输入，而共享对象可以由任何交易使用，但只能具有指定的可变权限。
See Section 4.4 for a full explanation.
完整解释见第4.4节。

There are two types of objects: package code objects, and struct data objects. A package object contains of a list of modules. A struct object contains a Move struct value and the Move type of that value.
有两种类型的对象：包代码对象和结构数据对象。包对象包含一个模块列表。结构对象包含一个Move结构值和该值的Move类型。
The contents of an object may change, but its ID, object type (package vs struct) and Move struct type are immutable. This ensures that objects are strongly typed and have a persistent identity.
对象的内容可能会更改，但其ID、对象类型（package vs struct）和Move struct类型是不可变的。这样可以确保对象是强类型的并且具有持久标识。

Finally, an object contains a version. Freshly created objects have version 0, and an object’s version is incremented each time a transaction takes the object as an input.
最后，一个对象包含一个版本。新创建的对象的版本为0，每次事务将对象作为输入时，对象的版本都会递增。

##3.4 Addresses and Authenticators(地址和身份验证人)

Authenticator = Ed25519PubKey ⊎ ECDSAPubKey ⊎ . . .
Addr          = 𝐶𝑜𝑚(Authenticator)

Table 4: Addresses and Authenticators

An address is the persistent identity of a Sui end-user (although note that a single user can have an arbitrary number of addresses).
地址是Sui最终用户的持久身份（尽管请注意，单个用户可以有任意数量的地址）。
To transfer an object to another user, the sender must know the address of the recipient.
若要将对象传输给另一个用户，发件人必须知道收件人的地址。

As we will discuss shortly, a Sui transaction must contain the address of the user sending (i.e., initiating) the transaction and an authenticator whose digest matches the address. The separation between addresses and authenticators enables cryptographic agility.
正如我们稍后将要讨论的那样，Sui交易必须包含发送（即启动）交易的用户的地址以及摘要与地址匹配的验证器。地址和验证器之间的分离实现了加密的灵活性。
An authenticator can be a public key from any signature scheme, even if the schemes use different key lengths (e.g., to support postquantum signatures). In addition, an authenticator need not be a single public key–it could also be (e.g.) a K-of-N multisig key.
验证器可以是来自任何签名方案的公钥，即使这些方案使用不同的密钥长度（例如，支持量子后签名）。此外，验证器不需要是单个公钥，也可以是（例如）N个多签名密钥中的K个。

##3.5 Transactions(交易)

ObjRef     = ObjID × Version × 𝐶𝑜𝑚(Obj)
CallTarget = ObjRef × ModuleName × FunName
CallArg    = ObjRef ⊎ ObjID ⊎ PrimType
Package    = [Module]
Publish    = Package × [ObjRef]
Call       = CallTarget × [StorableType] × [CallArg]
GasInfo    = ObjRef × MaxGas × BaseFee × Tip
Tx         = (Call ⊎ Publish) × GasInfo × Addr × Authenticator

Table 5: Transactions

Sui has two different transaction types: publishing a new Move package, and calling a previously published Move package. A publish transaction contains a package–a set of modules that will be published together as a single object, as well as the dependencies of all the modules in this package (encoded as a list of object references that must refer to already-published package objects).
Sui有两种不同的交易类型：发布新的Move包和调用以前发布的Move包。发布交易包含一个包——一组将作为单个对象一起发布的模块，以及该包中所有模块的依赖项（编码为必须引用已发布的包对象的对象引用列表）。
To execute a publish transaction, the Sui runtime will run the Move bytecode verifier on each package, link the package against its dependencies, and run the module initializer of each module. Module initializers are useful for bootstrapping the initial state of an application implemented by the package.
要执行一个发布交易，Sui运行时将在每个包上运行Move字节码验证器，根据包的依赖项链接包，并运行每个模块的模块初始化器。模块初始化程序对于引导包实现的应用程序的初始状态很有用。

A call transaction’s most important arguments are object inputs. Object arguments are either specified via an object reference (for single-owner and shared immutable objects) or an object ID (for shared mutable objects).
调用交易最重要的参数是对象输入。对象参数可以通过对象引用（对于单个所有者和共享的不可变对象）或对象ID（对于共享的可变对象）指定。
An object reference consists of an object ID, an object version, and the hash of the object value. The Sui runtime will resolve both object ID’s and object references to object values stored in the global object pool.
对象引用由对象ID、对象版本和对象值的哈希组成。Sui运行时将解析对象ID和对全局对象池中存储的对象值的对象引用。
For object references, the runtime will check the version of the reference against the version of the object in the pool, as well as checking that the reference’s hash matches the pool object. This ensures that the runtime’s view of the object matches the transaction sender’s view of the object.
对于对象引用，运行时将对照池中对象的版本检查引用的版本，并检查引用的哈希是否与池对象匹配。这样可以确保运行时的对象视图与事务发送方的对象视图相匹配。

In addition, a call transaction accepts type arguments and pure value arguments. Type arguments instantiate generic type parameters of the entrypoint function to be invoked (e.g., if the entrypoint function is send_coin<T>(c: Coin<T>, ...), the generic type parameter T could be instantiated with the type argument SUI to send the Sui native token). Pure values can include primitive types and vectors of primitive types, but not struct types.
此外，调用交易接受类型参数和纯值参数。类型参数实例化要调用的入口点函数的通用类型参数（例如，如果入口点函数是 send_coin<T>(c: Coin<T>, ...)，通用类型参数 T 可以用类型参数实例化SUI 发送 Sui 本地令牌）。纯值可以包括基本类型和基本类型的向量，但不包括结构类型。

The function to be invoked by the call is specified via an object reference (which must refer to a package object), a name of a module in that package, and a name of a function in that package.
调用要调用的函数是通过对象引用（必须引用包对象）、该包中模块的名称以及该包中函数的名称指定的。
To execute a call transaction, the Sui runtime will resolve the function, bind the type, object, and value arguments to the function parameters, and use the Move VM to execute the function.
为了执行调用交易，Sui 运行时将解析函数，将类型、对象和值参数绑定到函数参数，并使用 Move VM 来执行函数。

Both call and publish transactions are subject to gas metering and gas fees. The metering limit is expressed by a maximum gas budget. The runtime will execute the transaction until the budget is reached, and will abort with no effects (other than deducting fees and reporting the abort code) if the budget is exhausted.
调用和发布交易都需要缴纳 gas 计量和 gas 费用。计量限制由最大 gas 预算表示。运行时将执行交易，直到达到预算，如果预算用尽，将中止并且没有任何影响（除了扣除费用和报告中止代码）。

The fees are deducted from a gas object specified as an object reference. This object must be a Sui native token (i.e., its type must be Coin<SUI>).
费用从指定为对象引用的gas对象中扣除。此对象必须是 Sui 原生令牌（即，其类型必须是 Coin<SUI>）。
Sui uses EIP15594 -style fees: the protocol defines a base fee (denominated in gas units per Sui token) that is algorithmically adjusted at epoch boundaries, and the transaction sender can also include an optional tip (denominated in Sui tokens).
Sui 使用 EIP15594 类型的费用：该协议定义了一个基本费用（以每个 Sui 代币的gas单位计价），该费用在纪元边界通过算法进行调整，并且交易发送方还可以包括一个可选的小费（以 Sui 代币计价）。
Under normal system load, transactions will be processed promptly even with no tip. However, if the system is congested, transactions with a larger tip will be prioritized. The total fee deduced from the gas object is (GasUsed ∗ BaseFee) + Tip.
在正常的系统负载下，即使没有提示，交易也会迅速处理。但是，如果系统拥塞，小费较大的交易将被优先处理。从 gas 对象中扣除的总费用为 (GasUsed * BaseFee) + Tip。

##3.6 Transaction Effects(交易影响)

Event          = StructType × Struct
Create         = Obj
Update         = Obj
Wrap           = ObjID × Version
Delete         = ObjID × Version
ObjEffect      = Create ⊎ Update ⊎ Wrap ⊎ Delete
AbortCode      = N × ModuleName
SuccessEffects = [ObjEffect] × [Event]
AbortEffects   = AbortCode
TxEffects      = SuccessEffects ⊎ AbortEffects

Table 6: Transaction Effects

Transaction execution generates transaction effects which are different in the case when execution of a transaction is successful(SuccessEffects in Table 6) and when it is not (AbortEffects in Table 6).
交易执行产生的交易效果在交易执行成功时（表 6 中的 SuccessEffects）和失败时（表 6 中的 AbortEffects）是不同的。

Upon successful transaction execution, transaction effects include information about changes made to Sui’s global object pool(including both updates to existing objects and freshly created objects) and events generated during transaction execution.
交易执行成功后，交易影响包括对 Sui 的全局对象池所做的更改（包括对现有对象和新创建对象的更新）和交易执行期间生成的事件的信息。
Another effect of successful transaction execution could be object removal(i.e., deletion) from the global pool and also wrapping (i.e., embedding) one object into another, which has a similar effect to removal – a wrapped object disappears from the global pool and exists only as a part of the object that wraps it. Since deleted and wrapped objects are no longer accessible in the global pool, these effects are represented by the ID and version of the object.
交易执行成功的另一个影响可能是对象从全局池中移除（即删除）并将一个对象包装（即嵌入）到另一个对象中，这与移除具有类似的效果——一个被包装的对象从全局池中消失并存在仅作为包装它的对象的一部分。由于已删除和包装的对象在全局池中不再可访问，因此这些影响由对象的 ID 和版本表示。

Events encode side effects of successful transaction execution beyond updates to the global object pool. Structurally, an event consists of a Move struct and its type. Events are intended to be consumed by actors outside the blockchain, but cannot be read by Move programs.
除了对全局对象池的更新之外，交易还会对成功执行交易的副作用进行编码。从结构上讲，事件由 Move 结构及其类型组成。事件旨在供区块链外部的参与者使用，但不能被 Move 程序读取。

Transactions in Move have an all-or-nothing semantics – if execution of a transaction aborts at some point (e.g., due to an unexpected failure), even if some changes to objects had happened(or some events had been generated) prior to this point, none of these effects persist in an aborted transaction. Instead, an aborted transaction effect includes a numeric abort code and the name of a module where the transaction abort occurred. Gas fees are still charged for aborted transactions.
Move 中的交易具有全有或全无的语义——如果交易的执行在某个时刻中止（例如，由于意外故障），即使在此之前对象发生了一些更改（或生成了一些事件）一点，这些影响都不会在中止的交易中持续存在。相反，中止的交易效果包括数字中止代码和发生交易中止的模块的名称。对于中止的交易，仍会收取gas费。

#4 THE SUI SYSTEM(SUI系统)

In this section we describe Sui from a systems’ perspective, including the mechanisms to ensure safety and liveness across authorities despite Byzantine failures. We also explain the operation of clients, including light clients that need some assurance about the system state without validating its full state.
在本节中，我们从系统的角度描述了 Sui，包括在拜占庭式失败的情况下确保跨当局安全和活跃的机制。我们还解释了客户端的操作，包括需要在不验证其完整状态的情况下保证系统状态的轻客户端。

Brief background. At a systems level Sui is an evolution of the FastPay [3] low-latency settlement system, extended to operate on arbitrary objects through user-defined smart contracts, and with a permissionless delegated proof of stake committee composition [2].
简要背景。在系统层面上，Sui 是 FastPay [3] 低延迟结算系统的演变，通过用户定义的智能合约扩展到对任意对象进行操作，并具有无许可委托的权益委员会组成证明 [2]。
Basic asset management by object owners is based on a variant of Byzantine consistent broadcast [6] that has lower latency and is easier to scale across many machines as compared to traditional implementations of Byzantine consensus [8, 11, 12].When full agreement is required we use a high-throughput DAG-based consensus, e.g. [9] to manage locks, while execution on different shared objects is parallelized.
对象所有者的基本资产管理基于拜占庭一致性广播 [6] 的变体，与拜占庭共识的传统实现方式 [8、11、12] 相比，它具有更低的延迟并且更容易在多台机器上扩展。当完全一致时要求我们使用基于 DAG 的高吞吐量共识，例如[9] 来管理锁，同时并行执行不同的共享对象。

Protocol outline. Figure 1 illustrates the high-level interactions between a client and Sui authorities to commit a transaction. We describe them here briefly:
协议大纲。图 1 说明了客户与 Sui 当局之间为提交交易而进行的高级交互。我们在这里简要描述它们：

• A user with a private signing key creates and signs a user transaction to mutate objects they own, or shared objects, within Sui. Subsequently, user signature keys are not needed, and the remaining of the process may be performed by the user client, or a gateway on behalf of the user (denoted as keyless operation in the diagram).
• 拥有私有签名密钥的用户创建并签署用户交易，以在 Sui 中改变他们拥有的对象或共享的对象。随后，不需要用户签名密钥，剩下的过程可以由用户客户端或网关代表用户执行（图中表示为无密钥操作）。

• The user transaction is sent to the Sui authorities, that each check it for validity, and upon success sign it and return the signed transaction to the client. The client collects the responses from a quorum of authorities to form a transaction certificate.
• 用户交易被发送到Sui 当局，每个当局检查它的有效性，并在成功后签署并将签署的交易返回给客户。客户端收集来自法定机构的响应以形成交易证书。

• The transaction certificate is then sent back to all authorities, and if the transaction involves shared objects it is also sent to a Byzantine agreement protocol operated by the Sui authorities. Authorities check the certificate, and in case shared objects are involved also wait for the agreement protocol to sequence it in relation to other shared object transactions, and then execute the transaction and summarize its effects into a signed effects response.
• 然后将交易证书发送回所有当局，如果交易涉及共享对象，它也会发送到由 Sui 当局运行的拜占庭协议协议。当局检查证书，如果涉及共享对象，还等待协议将其与其他共享对象交易相关联地排序，然后执行交易并将其影响汇总到已签名的效果响应中。

• Once a quorum of authorities has executed the certificate its effects are final (denoted as finality in the diagram). Clients can collect a quorum of authority responses and create an effects certificate and use it as a proof of the finality of the transactions effects.
• 一旦权威机构的法定人数执行了证书，其效果就是最终的（在图中表示为最终性）。客户可以收集法定人数的权威响应并创建效果证书并将其用作交易效果最终性的证明。

![avatar](./Figure-1.jpg)

This section describes each of these operations in detail, as well as operations to reconfigure and manage state across authorities.
本节详细描述了这些操作中的每一个，以及跨权限重新配置和管理状态的操作。

##4.1 System Model(系统模型)

Sui operates in a sequence of epochs denoted by 𝑒 ∈ {0, . . .}. Each epoch is managed by a committee 𝐶𝑒 = (𝑉𝑒, 𝑆𝑒 (·)), where 𝑉𝑒 is a set of authorities with known public verification keys and network end-points.
Sui 操作由 𝑒 ∈ {0, . . .}代表的一系列世代表示。每个世代由委员会𝐶𝑒 = (𝑉𝑒, 𝑆𝑒 (·)) 管理，其中𝑉𝑒 是一组具有已知公共验证密钥和网络端点的权限。
The function 𝑆𝑒 (𝑣) maps each authority 𝑣 ∈ 𝑉𝑒 to a number of units of delegated stake. We assume that 𝐶𝑒 for each epoch is signed by a quorum (see below) of authority stake at epoch 𝑒 −1. (Sect. 4.7 discusses the formation and management of committees). Within an epoch, some authorities are correct (they follow the protocol faithfully and are live), while others are Byzantine (they deviate arbitrarily from the protocol). The security assumption is that the set of honest authorities 𝐻𝑒 ⊆ 𝑉𝑒 is assigned a quorum of stake within the epoch, i.e. Í ℎ∈𝐻𝑒 𝑆𝑒 (ℎ) > 2/3 Í 𝑣∈𝑉𝑒 𝑆𝑒 (𝑣) (and refer to any set of authorities with over two-thirds stake as a quorum).
函数 𝑆𝑒 (𝑣) 将每个权限 𝑣 ∈ 𝑉𝑒 映射到许多委托权益单位。我们假设每个世代的𝐶𝑒 由世代𝑒 -1 的法定权益的法定人数（见下文）签署。 （第 4.7 节讨论了委员会的组建和管理）。在一个世代内，一些权威是正确的（他们忠实地遵循协议并且是活的），而另一些是拜占庭的（他们任意偏离协议）。安全假设是诚实权威集合 𝐻𝑒 ⊆ 𝑉𝑒 被分配了一个世代内的法定股权，即 Í ℎ∈𝐻𝑒 𝑆𝑒 (ℎ) > 2/3 Í 𝑣∈𝑉𝑒 𝑆𝑒 (𝑣)（并参考任何集合超过三分之二的股份作为法定人数的当局）。

There exists at least one live and correct party that acts as a relay for each certificate (see Sect. 4.3) between honest authorities. This ensures liveness, and provides an eventual delivery property to the Byzantine broadcast (see totality of reliable broadcast in [6]).
至少存在一个有效且正确的一方，作为诚实当局之间每个证书的中继（见第 4.3 节）。这确保了活跃性，并为拜占庭广播提供了最终的交付属性（参见 [6] 中可靠广播的完整性）。
Each authority operates such a relay, either individually or through a collective dissemination protocol. External entities, including Sui light clients, replicas and services may also take on this role. The distinction between the passive authority core, and an internal or external active relay component that is less reliable or trusted, ensures a clear demarcation and minimization of the Trusted Computing Base [15] on which Sui’s safety and liveness relies.
每个机构都可以单独或通过集体传播协议来操作这样的中继。外部实体，包括 Sui 轻客户端、副本和服务，也可以承担这个角色。被动授权核心与不太可靠或不可信的内部或外部主动中继组件之间的区别，确保了 Sui 的安全性和活跃性所依赖的可信计算库 [15] 的明确划分和最小化。

##4.2 Authority & Replica Data Structures(权限和副本数据结构)

Sui authorities rely on a number of data structures to represent state. We define these structures based on the operations they support. They all have a deterministic byte representation.
Sui 权限依靠一些数据结构来表示状态。我们根据它们支持的操作来定义这些结构。它们都有确定的字节表示。

An Object (Obj) stores user smart contracts and data within Sui. They are the Sui system-level encoding of the Move objects introduced in Sect. 2. They support the following set of operations:
一个对象（Obj）在 Sui 中存储用户智能合约和数据。它们是 第2节 中介绍的 Move 对象的 Sui 系统级编码。它们支持以下一组操作：

• ref(Obj) returns the reference (ObjRef) of the object, namely a triplet (ObjID, Version, ObjDigest). ObjID is practically unique for all new objects created, and Version is an increasing positive integer representing the object version as it is being mutated.
• ref(Obj) 返回对象的引用(ObjRef)，即三元组(ObjID、Version、ObjDigest)。 ObjID 对于所有创建的新对象来说实际上是唯一的，而 Version 是一个递增的正整数，表示正在发生变化的对象版本。

• owner(Obj) returns the authenticator Auth of the owner of the object. In the simplest case, Auth is an address, representing a public key that may use this object. More complex authenticators are also available (see Sect. 4.4).
• owner(Obj) 返回对象所有者的身份验证器Auth。在最简单的情况下，Auth 是一个地址，代表一个可以使用这个对象的公钥。也可以使用更复杂的验证器（参见第 4.4 节）。

• read-only(Obj) returns true if the object is read-only. Readonly objects may never be mutated, wrapped or deleted. They may also be used by anyone, not just their owners.
• 如果对象是只读的，则只读(Obj) 返回真。只读对象可能永远不会被改变、包装或删除。它们也可以被任何人使用，而不仅仅是它们的所有者。

• parent(Obj) returns the transaction digest (TxDigest) that last mutated or created the object.
• parent(Obj) 返回最后一次改变或创建对象的交易摘要（TxDigest）。

• contents(Obj) returns the object type Type and data Data that can be used to check the validity of transactions and carry the application-specific information of the object.
• contents(Obj) 返回对象类型Type 和数据Data，可用于检查交易的有效性并携带对象的应用程序特定信息。

The object reference (ObjRef) is used to index objects. It is also used to authenticate objects since ObjDigest is a commitment to their full contents.
对象引用 (ObjRef) 用于索引对象。它也用于验证对象，因为 ObjDigest 是对其全部内容的承诺。

A transaction (Tx) is a structure representing a state transition for one or more objects. They support the following set of operations:
交易 (Tx) 是表示一个或多个对象的状态转换的结构。它们支持以下一组操作：

• digest(Tx) returns the TxDigest, which is a binding cryptographic commitment to the transaction.
• digest(Tx) 返回TxDigest，这是对交易的绑定加密承诺。

• epoch(Tx) returns the EpochID during which this transaction may be executed.
• epoch(Tx) 返回 EpochID，在此期间可能会执行此交易。

• inputs(Tx) returns a sequence of object [ObjRef] the transaction needs to execute.
• inputs(Tx) 返回交易需要执行的对象序列[ObjRef]。

• payment(Tx) returns a reference to an ObjRef to be used to pay for gas, as well as the maximum gas limit, and a conversion rate between a unit of gas and the unit of value in the gas payment object.
• payment(Tx) 返回对用于支付gas 的ObjRef 的引用，以及最大gas 限额，以及gas 支付对象中gas 单位与价值单位之间的转换率。

• valid(Tx, [Obj]) returns true if the transaction is valid, given the requested input objects provided. Validity is discussed in Sect. 4.4, and relates to the transactions being authorized to act on the input objects, as well as sufficient gas being available to cover the costs of its execution.
• 如果交易有效，valid(Tx, [Obj]) 返回真，给定所提供的请求输入对象。有效性在 Sect. 4.4，并涉及被授权对输入对象采取行动的交易，以及足够的 gas 可用于支付其执行成本。

• exec(Tx, [Obj]) executes the transaction and returns a structure Effects representing its effects. A valid transaction execution is infallible, and its output is deterministic.
• exec(Tx, [Obj]) 执行交易并返回表示其效果的结构Effects。有效的交易执行是绝对可靠的，其输出是确定性的。

A transaction is indexed by its TxDigest, which may also be used to authenticate its full contents. All valid transactions (except the special hard-coded genesis transaction) have at least one owned input, namely the objects used to pay for gas.
交易由其 TxDigest 索引，也可用于验证其全部内容。所有有效交易（特殊的硬编码创世交易除外）至少有一个拥有的输入，即用于支付 gas 的对象。

A transaction effects (Effects) structure summarizes the outcome of a transaction execution. It supports the following operations:
一个交易效果（Effects）结构总结了一个交易执行的结果。它支持以下操作：

• digest(Effects) is a commitment EffDigest to the Effects structure, that may be used to index or authenticate it.
• digest(Effects) 是EffDigest 对Effects 结构的承诺，可用于索引或验证它。

• transaction(Effects) returns the TxDigest of the executed transaction yielding the effects.
• transaction(Effects) 返回产生效果的已执行交易的TxDigest。

• dependencies(Effects) returns a sequence of dependencies[TxDigest] that should be executed before the transaction with these effects may execute.
• dependencies(Effects) 返回一系列dependencies[TxDigest]，这些dependencies[TxDigest] 应该在具有这些影响的交易执行之前执行。

• contents(Effects) returns a summary of the execution. Status reports the outcome of the smart contract execution. The lists Created, Mutated, Wrapped, Unwrapped and Deleted, list the object references that underwent the respective operations. And Events lists the events emitted by the execution.
• contents(Effects) 返回执行摘要。状态报告智能合约执行的结果。 Created、Mutated、Wrapped、Unwrapped 和 Deleted 列表列出了经过相应操作的对象引用。 Events 列出了执行发出的事件。

A transaction certificate TxCert on a transaction contains the transaction itself as well as the identifiers and signatures from a quorum of authorities.
交易的交易证书 TxCert 包含交易本身以及来自法定机构的标识符和签名。
Note that a certificate may not be unique, in that the same logical certificate may be represented by a different set of authorities forming a quorum.
请注意，证书可能不是唯一的，因为相同的逻辑证书可能由形成法定人数的一组不同的授权机构表示。
Additionally, a certificate might not strictly be signed by exactly a 2/3 quorum, but possibly more if more authorities are responsive. However, two different valid certificates on the same transaction should be treated as representing semantically the same certificate.
此外，证书可能不会严格由 2/3 的法定人数签署，但如果有更多的授权机构响应，则可能更多。但是，同一交易中的两个不同的有效证书应被视为在语义上表示相同的证书。
A partial certificate (TxSign) contains the same information, but signatures from a set of authorities representing stake lower than the required quorum, usually a single authority.
部分证书 (TxSign) 包含相同的信息，但来自一组权限的签名代表低于所需的法定人数，通常是单个权限。
The identifiers of signers are included in the certificate (i.e., accountable signatures [? ]) to identify authorities ready to process the certificate, or that may be used to download past information required to process the certificate (see Sect. 4.8).
签名者的标识符包含在证书中（即负责签名 [?]），以识别准备好处理证书的机构，或者可用于下载处理证书所需的过去信息（参见第 4.8 节）。

Similarly, an effects certificate EffCert on an effects structure contains the effects structure itself, and signatures from authorities5 that represent a quorum for the epoch in which the transaction is valid.
类似地，效果结构上的效果证书 EffCert 包含效果结构本身，以及来自权威机构的签名5，代表交易有效时期的法定人数。
The same caveats, about non-uniqueness and identity apply as for transaction certificates. A partial effects certificate, usually containing a single authority signature and the effects structure is denoted as EffSign.
关于非唯一性和身份的相同警告适用于交易证书。部分效果证书，通常包含单个授权机构签名，效果结构表示为 EffSign。

Persistent Stores. Each authority and replica maintains a set of persistent stores. The stores implement persistent map semantics and can be represented as a set of key-value pairs (denoted 𝑚𝑎𝑝 [𝑘𝑒𝑦] → 𝑣𝑎𝑙𝑢𝑒), such that only one pair has a given key.
持久性存储。每个授权机构和复制副本都维护一组持久存储。存储实现持久映射语义，并且可以表示为一组键值对（表示为𝑚𝑎𝑝 [𝑘𝑒𝑦] → 𝑣𝑎𝑙𝑢𝑒), 使得只有一对具有给定的密钥。
Before a pair is inserted a contains(𝑘𝑒𝑦) call returns false, and get(𝑘𝑒𝑦)returns an error. After a pair is inserted contains(𝑘𝑒𝑦) calls returns true, and get(𝑘𝑒𝑦) return the value. An authority maintains the following persistent stores:
在插入一对之前，包含(𝑘𝑒𝑦) 调用返回false，并获取(𝑘𝑒𝑦)返回一个错误。插入一对后包含(𝑘𝑒𝑦) calls返回true，得到(𝑘𝑒𝑦) 返回值。授权机构维护以下持久存储：

• The order lock map Lock𝑣 [ObjRef] → TxSignOption records the first valid transaction Tx seen and signed by the authority for an owned object version ObjRef, or None if the object version exists but no valid transaction using as an input it has been seen. It may also record the first certificate seen with this object as an input. This table, and its update rules, represents the state of the distributed locks on objects across Sui authorities, and ensures safety under concurrent processing of transactions.
• 订单锁定map Lock𝑣 [对象参考]→ TxSignOption记录由拥有的对象版本ObjRef的授权机构看到并签署的第一个有效交易Tx，或者如果对象版本存在但没有看到有效交易作为输入，则记录None。它还可以记录使用该对象作为输入看到的第一个证书。该表及其更新规则代表了跨Sui权限的对象上的分布式锁的状态，并确保了事务并发处理下的安全性。

• The certificate map Ct𝑣 [TxDigest] → (TxCert, EffSign)records all full certificates TxCert, which also includes Tx, processed by the authority within their validity epoch, along with their signed effects EffSign. They are indexed by transaction digest TxDigest
• 证书map Ct𝑣 [Tx摘要]→ （TxCert，EffSign）记录所有完整证书TxCert（也包括Tx），由权威机构在其有效期内处理，以及其签名效果EffSign。它们按事务摘要TxDigest进行索引

• The object map Obj𝑣 [ObjRef] → Obj records all objects Obj created by transactions included in certificates within Ct𝑣 indexed by ObjRef. This store can be completely derived by re-executing all certificates in Ct𝑣 . A secondary index is maintained that maps ObjID to the latest object with this ID. This is the only information necessary to process new transactions, and older versions are only maintained to facilitate reads and audit.
• 对象map Obj𝑣 [对象参考]→ Obj记录Ct𝑣中证书中包含的交易创建的所有对象Obj𝑣 由ObjRef索引。通过重新执行Ct𝑣中的所有证书，可以完全派生此存储𝑣 . 维护一个辅助索引，将ObjID映射到具有此ID的最新对象。这是处理新事务所需的唯一信息，而维护旧版本只是为了方便读取和审核。

• The synchronization map Sync𝑣 [ObjRef] → TxDigest indexes all certificates within Ct𝑣 by the objects they create, mutate or delete as tuples ObjRef. This structure can be fully re-created by processing all certificates in Ct𝑣 , and is used to help client synchronize transactions affecting objects they care about.
• 同步map Sync𝑣 [对象参考]→ TxDigest对Ct中的所有证书进行索引𝑣 通过它们作为元组ObjRef创建、变异或删除的对象。通过处理Ct中的所有证书，可以完全重新创建此结构𝑣 , 并且用于帮助客户端同步影响他们关心的对象的事务。

Authorities maintain all four structures, and also provide access to local checkpoints of their certificate map to allow other authorities and replicas to download their full set of processed certificates.
授权机构维护所有四个结构，并提供对其证书映射的本地检查点的访问，以允许其他授权机构和副本下载他们的全套已处理证书。
A replica does not process transactions but only certificates, and re-executes them to update the other tables as authorities do. It also maintains an order lock map to audit non-equivocation.
副本不处理交易，只处理证书，并像授权机构一样重新执行它们以更新其他表。它还维护一个订单锁映射以审核非模棱两可的情况。

An authority may be designed as a full replica maintaining all four stores (and checkpoints) to facilitate reads and synchronization, combined with a minimal authority core that only maintains object locks and objects for the latest version of objects used to process new transactions and certificates. This minimizes the Trusted Computing Base relied upon for safety.
授权可以设计为维护所有四个存储（和检查点）以促进读取和同步的完整副本，并结合最小授权核心，该核心仅维护对象锁和用于处理新交易和证书的最新版本对象的对象。这最大限度地减少了安全所依赖的可信计算库。

Only the order lock map requires strong key self-consistency, namely a read on a key should always return whether a value or None is present for a key that exists, and such a check should be atomic with an update that sets a lock to a non-None value.
只有 order lock map 需要强键自洽性，即对键的读取应该始终返回是否存在值或 None 对于存在的键，并且这样的检查应该是原子的，更新将锁设置为非无值。
This is a weaker property than strong consistency across keys, and allows for efficient sharding of the store for scaling. The other stores may be eventually consistent without affecting safety.
这是一个比键之间的强一致性更弱的属性，并且允许对存储进行有效分片以进行扩展。其他存储可能在不影响安全的情况下最终保持一致。

##4.3 Authority Base Operation(权限库操作)

Process Transaction. Upon receiving a transaction Tx an authority performs a number of checks:
处理交易。收到交易 Tx 后，权威机构会执行多项检查：

(1) It ensures epoch(Tx) is the current epoch.
(1) 它确保 epoch(Tx) 是当前 epoch。
(2) It ensures all object references inputs(Tx) and the gas object reference in payment(Tx) exist within Obj𝑣 and loads them into [Obj]. For owned objects the exact reference should be available; for read-only or shared objects the object ID should exist.
(2) 它确保所有对象引用 inputs(Tx) 和 payment(Tx) 中的 gas 对象引用存在于 Obj𝑣 中，并将它们加载到 [Obj] 中。对于拥有的对象，应该有准确的引用；对于只读或共享对象，对象 ID 应该存在。
(3) Ensures sufficient gas can be made available in the gas object to cover the cost of executing the transaction.
(3) 确保 gas 对象中有足够的 gas 可用以支付执行交易的成本。
(4) It checks valid(Tx, [Obj]) is true. This step ensures the authentication information in the transaction allows access to the owned objects.
(4) 它检查 valid(Tx, [Obj]) 是否为真。此步骤确保事务中的身份验证信息允许访问拥有的对象。
(5) It checks that Lock𝑣 [ObjRef] for all owned inputs(Tx) objects exist, and it is either None or set to the same Tx, and atomically sets it to TxSign. (We call these the ‘locks checks’).
(5) 它检查所有拥有的 inputs(Tx) 对象的 Lock𝑣 [ObjRef] 是否存在，并且它是 None 或设置为相同的 Tx，并自动将其设置为 TxSign。 （我们称这些为“锁检查”）。

If any of the checks fail processing ends, and an error is returned. However, it is safe for a partial update of Lock𝑣 to persist (although our current implementation does not do partial updates, but atomic updates of all locks).
如果任何检查失败，处理结束，并返回错误。但是，Lock𝑣 的部分更新持久化是安全的（尽管我们当前的实现不进行部分更新，而是对所有锁进行原子更新）。

If all checks are successful then the authority returns a signature on the transaction, ie. a partial certificate TxSign. Processing an order is idempotent upon success, and returns a partial certificate(TxSign), or a full certificate (TxCert) if one is available.
如果所有检查都成功，则权威机构返回交易签名，即部分证书 TxSign。成功处理订单是幂等的，并返回部分证书（TxSign）或完整证书（TxCert）（如果可用）。

Any party may collate a transaction and signatures (TxSign)for a set of authorities forming a quorum for epoch 𝑒, to form a transaction certificate TxCert.
任何一方都可以整理交易和一组授权机构的签名（TxSign），形成世代 𝑒 的法定人数，以形成交易证书 TxCert。

Process Certificate. Upon receiving a certificate an authority checks all validity conditions for the transaction, except those relating to locks (the so-called ‘locks checks’). Instead it performs the following checks: for each owned input object in inputs(Tx) it checks that the lock exists, and that it is either None, set to any TxSign, or set to a certificate for the same transaction as the current certificate.
过程证书。收到证书后，权威机构会检查交易的所有有效性条件，但与锁相关的条件除外（所谓的“锁检查”）。相反，它执行以下检查：对于 inputs(Tx) 中的每个拥有的输入对象，它检查锁是否存在，以及它是否为 None，设置为任何 TxSign，或设置为与当前证书相同的事务的证书。
If this modified locks check fails, the authority has detected an unrecoverable Byzantine failure, halts normal operations, and starts a disaster recovery process. For shared objects (see Sect. 4.4) authorities check that the locks have been set through the certificate being sequenced in a consensus, to determine the version of the share object to use. If so, the transaction may be executed; otherwise it needs to wait for such sequencing first.
如果此修改后的锁检查失败，则权威机构检测到不可恢复的拜占庭故障，停止正常操作，并启动灾难恢复过程。对于共享对象（参见第 4.4 节），当局检查是否已通过在共识中排序的证书设置了锁，以确定要使用的共享对象的版本。如果是，则可以执行交易；否则需要先等待这样的排序。

If the check succeeds, the authority adds the certificate to its certificate map, along with the effects resulting from its execution, ie. Ct𝑣 [TxDigest] → (TxCert, EffSign); it updates the locks map to record the certificate Lock𝑣 [ObjRef] → TxCert for all owned input objects that have locks not set to a certificate.
如果检查成功，权威机构将证书添加到其证书映射中，以及其执行产生的效果，即。 Ct𝑣 [TxDigest] → (TxCert, EffSign);它更新锁集合以记录证书 Lock𝑣 [ObjRef] → TxCert 用于所有拥有未设置为证书的锁的输入对象。
As soon as all objects in Input(Tx) is inserted in Obj𝑣 , then all effects in EffSign are also materialized by adding their ObjRef and contents to Obj𝑣 . Finally for all created or mutated in EffSign the synchronization map is updated to map them to Tx.
一旦 Input(Tx) 中的所有对象都插入到 Obj𝑣 中，然后 EffSign 中的所有效果也会通过将它们的 ObjRef 和内容添加到 Obj𝑣 中来具体化。最后，对于在 EffSign 中创建或更改的所有内容，同步映射将更新以将它们映射到 Tx。

Remarks. The logic for handling transactions and certificates leads to a number of important properties:
重点。处理交易和证书的逻辑导致了许多重要的属性：

• Causality & parallelism. The processing conditions for both transactions and certificates ensure causal execution:an authority only ‘votes’ by signing a transaction if it has processed all certificates creating the objects the transaction depends upon, both owned, shared and read-only. Similarly, an authority only processes a certificate if all input objects upon which it depends exist in its local objects map. This imposes a causal execution order, but also enables transactions not causally dependent on each other to be executed in parallel on different cores or machines.
• 因果关系和平行关系。交易和证书的处理条件确保了因果执行：如果一个机构已经处理了创建交易所依赖的对象的所有证书，则该机构仅通过签署交易来“投票”，包括拥有的、共享的和只读的。类似地，如果授权所依赖的所有输入对象都存在于其本地对象映射中，则授权仅处理证书。这强加了一个因果执行顺序，但也使相互不因果依赖的交易能够在不同的内核或机器上并行执行。

• Sign once, and safety. All owned input objects locks in Lock𝑣 [·] are set to the first transaction Tx that passes the checks using them, and then the first certificate that uses the object as an input. We call this locking the object to this transaction, and there is no unlocking within an epoch. As a result an authority only signs a single transaction per lock, which is an essential component of consistent broadcast [6], and thus the safety of Sui.
• 一次签名，安全。 Lock𝑣 [·] 中所有拥有的输入对象锁被设置为使用它们通过检查的第一个交易 Tx，然后是使用该对象作为输入的第一个证书。我们把这个对象锁定到这个交易上，一个epoch内是不会解锁的。因此，权威机构每个锁只签署一个交易，这是一致广播 [6] 的重要组成部分，因此也是 Sui 的安全性。

• Disaster recovery. An authority detecting two contradictory certificates for the same lock, has proof of irrecoverable Byzantine behaviour – namely proof that the quorum honest authority assumption does not hold. The two contradictory certificates are a fraud proof [1], that may be shared with all authorities and replicas to trigger disaster recovery processes. Authorities may also get other forms of proof of unrecoverable byzantine behaviour such as >1/3 signatures on effects (EffSign) that represent an incorrect execution of a certificate. Or a certificate with input objects that do not represent the correct outputs of previously processed certificates. These also can be packaged as a fraud proof and shared with all authorities and replicas. Note these are distinct from proofs that a tolerable minority of authorities(≤ 1/3 by stake) or object owners (any number) is byzantine or equivocating, which can be tolerated without any service interruption.
• 灾难恢复。为同一把锁检测到两个相互矛盾的证书的权威机构，拥有不可恢复的拜占庭行为的证据——即证明群体诚实权威假设不成立的证据。这两个相互矛盾的证书是欺诈证明 [1]，可以与所有授权机构和副本共享以触发灾难恢复过程。权威机构还可以获得其他形式的不可恢复的拜占庭行为证明，例如 >1/3 的效果签名 (EffSign)，表示证书执行不正确。或者带有输入对象的证书不代表先前处理的证书的正确输出。这些也可以打包为欺诈证明，并与所有权威机构和副本共享。请注意，这些与证明可容忍的少数权力机构（≤ 1/3 股份）或对象所有者（任何数量）是拜占庭式或模棱两可的证据不同，后者可以在没有任何服务中断的情况下被容忍。

• Finality. Authorities return a certificate (TxCert) and the signed effects (EffSign) for any read requests for an index in Lock𝑣 , Ct𝑣 and Obj𝑣 , Sync𝑣 . 
    A transaction is considered final if over a quorum of authorities reports Tx as included in their Ct𝑣 store. This means that an effects certificate(EffCert) is a transferable proof of finality. 
    However, a certificate using an object is also proof that all dependent certificates in its causal path are also final. Providing a certificate to any party, that may then submit it to a super majority of authorities for processing also ensures finality for the effects of the certificate. Note that finality is later than fastpay [3] to ensure safety under re-configuration.
    However, an authority can apply the effect of a transaction upon seeing a certificate rather than waiting for a commit.
• 终局性。权威机构为 Lock𝑣、Ct𝑣 和 Obj𝑣、Sync𝑣 中的索引的任何读取请求返回证书 (TxCert) 和签名效果 (EffSign)。
    如果超过法定人数的权威机构报告 Tx 包含在他们的 Ct𝑣 商店中，则交易被认为是最终的。这意味着效果证书（EffCert）是一种可转让的最终证明。
    但是，使用对象的证书也证明其因果路径中的所有相关证书也是最终的。向任何一方提供证书，然后可以将其提交给绝大多数权威机构进行处理，这也确保了证书效力的最终性。请注意，最终确定性晚于 fastpay [3] 以确保重新配置下的安全性。
    但是，授权机构可以在看到证书而不是等待提交时应用交易的影响。

##4.4 Owners, Authorization, and Shared Objects(所有者、授权和共享对象)

Transaction validity (see Sect. 4.3) ensures a transaction is authorized to include all specified input objects in a transaction. This check depends on the nature of the object, as well as the owner field.
交易有效性（见第 4.3 节）确保交易被授权在交易中包含所有指定的输入对象。此检查取决于对象的性质以及对象的所有者字段。

Read-only objects cannot be mutated or deleted, and can be used in transactions concurrently and by all users. Move modules for example are read-only. Such objects do have an owner that might be used as part of the smart contract, but that does not affect authorization to use them. They can be included in any transaction.
只读对象不能被修改或删除，并且可以在交易中并发地被所有用户使用。例如Move模块是只读的。这些对象确实有一个所有者，可以用作智能合约的一部分，但这并不影响使用它们的授权。它们可以包含在任何交易中。

Owned objects have an owner field. The owner can be set to an address representing a public key. In that case, a transaction is authorized to use the object, and mutate it, if it is signed by that address. A transaction is signed by a single address, and therefore can use one or more objects owned by that address.
拥有的对象有一个所有者字段。所有者可以设置为代表公钥的地址。在这种情况下，交易被授权使用该对象，如果它是由该地址签名的，则可以改变它。交易由单个地址签名，因此可以使用该地址拥有的一个或多个对象。
However, a single transaction cannot use objects owned by more than one address. The owner of an object, called a child object, can be set to the ObjID of another object, called the parent object.
但是，单个交易不能使用多个地址拥有的对象。一个对象（称为子对象）的所有者可以设置为另一个对象（称为父对象）的 ObjID。
In that case the child object may only be used if the parent object is included in the transaction, and the transaction is authorized to use the object. This facility may be used by contracts to construct efficient collections and other complex data structures.
在那种情况下，仅当父对象包含在交易中并且交易被授权使用该对象时，才可以使用子对象。合同可以使用此功能来构建高效的集合和其他复杂的数据结构。

Shared objects are mutable, but do not have a specific owner. They can instead be included in transactions by different parties, and do not require any authorization. Instead they perform their own authorization logic. Such objects, by virtue of having to support multiple writers while ensuring safety and liveness, do require a full agreement protocol to be used safely. Therefore they require additional logic before execution.
共享对象是可变的，但没有特定的所有者。相反，它们可以包含在不同方的交易中，并且不需要任何授权。相反，他们执行自己的授权逻辑。由于必须在确保安全性和活跃性的同时支持多个编写器，因此此类对象确实需要完整的协议协议才能安全使用。因此，它们在执行前需要额外的逻辑。
Authorities process transactions as specified in Sect. 4.3 for owned objects and read-only objects to manage their locks. However, authorities do not rely on consistent broadcast to manage the locks of shared objects. Instead, the creators of transactions that involve shared objects insert the certificate on the transaction into a high-throughput consensus system, e.g. [9].
权威机构按照 4.3节 中的规定处理交易。为自有对象和只读对象管理它们的锁。但是，权威机构不依赖一致的广播来管理共享对象的锁。相反，涉及共享对象的交易的创建者将交易证书插入高吞吐量共识系统，例如[9].
All authorities observe a consistent sequence of such certificates, and assign the version of shared objects used by each transaction according to this sequence. Then execution can proceed and is guaranteed to be consistent across all authorities. Authorities include the version of shared objects used in a transaction execution within the Effects certificate.
所有权威机构都遵守此类证书的一致顺序，并根据此顺序分配每个交易使用的共享对象的版本。然后执行可以继续，并保证在所有当局之间保持一致。授权包括在 Effects 证书中的交易执行中使用的共享对象的版本。

The above rules ensure that execution for transactions involving read-only and owned objects requires only consistent broadcast and a single certificate to proceed; and Byzantine agreement is only required for transactions involving shared objects.
上述规则确保涉及只读和拥有对象的交易的执行只需要一致的广播和单个证书即可进行；只有涉及共享对象的交易才需要拜占庭协议。
Smart contract authors can therefore design their types and their operations to optimize transfers and other operations on objects of a single user to have lower latency, while enjoying the flexibility of using shared objects to implement logic that needs to be accessed by multiple users.
因此，智能合约作者可以设计他们的类型和他们的操作来优化传输和单个用户对象的其他操作以具有更低的延迟，同时享受使用共享对象来实现需要由多个用户访问的逻辑的灵活性。

##4.5 Clients(客户端)

Full Clients & Replicas. Replicas, also sometimes called full clients, do not validate new transactions, but maintain a consistent copy of the valid state of the system for the purposes of audit, as well as to construct transactions or operate services incl. read infrastructures for light client queries.
完整的客户和副本。副本，有时也称为完整客户端，不验证新交易，但维护系统有效状态的一致副本以用于审计目的，以及构建交易或操作服务，包括。读取轻客户端查询的基础设施。

Light Clients. Both object references and transactions contain information that allows the authentication of the full causal chain of transactions that leading up to their creation or execution.
轻客户端。对象引用和交易都包含允许对导致其创建或执行的完整交易因果链进行身份验证的信息。
Specifically, an object reference (ObjRef) contains an ObjDigest that is an authenticator for the full state of the object, including the facility to get parent(Obj), namely the TxDigest that created the object. Similarly, a TxDigest authenticates a transaction, including the facility to extract through inputs(Tx) the object references of the input objects.
具体来说，对象引用 (ObjRef) 包含一个 ObjDigest，它是对象完整状态的验证器，包括获取父对象 (Obj) 的工具，即创建对象的 TxDigest。类似地，TxDigest 对交易进行身份验证，包括通过输入 (Tx) 提取输入对象的对象引用的功能。
Therefore the set of objects and certificates form a bipartite graph that is self-authenticating. Furthermore, effects structures are also signed, and may be collated into effects certificates that directly certify the results of transaction executions.
因此，对象和证书的集合形成了一个自认证的二分图。此外，效果结构也被签名，并且可以整理成效果证书，直接证明交易执行的结果。

These facilities may be used to support light clients that can perform high-integrity reads into the state of Sui, without maintaining a full replica node. Specifically an authority or full node may provide a succinct bundle of evidence, comprising a certificate TxCert on a transaction Tx and the input objects [Obj] corresponding to inputs(Tx) to convince a light client that a transition can take place within Sui.
这些设施可用于支持轻客户端，这些客户端可以执行对 Sui 状态的高完整性读取，而无需维护完整的副本节点。具体来说，权威机构或完整节点可以提供简洁的证据包，包括交易 Tx 上的证书 TxCert 和对应于输入（Tx）的输入对象 [Obj]，以说服轻客户端可以在 Sui 内进行转换。
A light client may then submit this certificate, or check whether it has been seen by a quorum or sample of authorities to ensure finality. Or it may craft a transaction using the objects resulting from the execution, and observe whether it is successful.
然后，轻客户端可以提交此证书，或检查它是否已被法定人数或权威样本看到以确保最终性。或者它可以使用执行产生的对象来创建一个交易，并观察它是否成功。

More directly, a service may provide an effects certificate to a client to convince them of the existence and finality of a transition within Sui, with no further action or interaction within the system.
更直接地说，服务可能会向客户提供效果证书，以说服他们在 Sui 中转换的存在和最终性，而无需在系统内进行进一步的操作或交互。
If a checkpoint of finalized certificates is available, at an epoch boundary or otherwise, a bundle of evidence including the input objects and certificate, alongside a proof of inclusion of the certificate in the checkpoint is also a proof of finality.
如果最终证书的检查点可用，在世代边界或其他地方，包括输入对象和证书在内的一揽子证据，以及证书包含在检查点中的证明也是最终证明。

Authorities may use a periodic checkpointing mechanism to create collective checkpoints of finalized transactions, as well as the state of Sui over time. A certificate with a quorum of stake over a checkpoint can be used by light clients to efficiently validate the recent state of objects and emitted events.
权威机构可以使用定期检查点机制来创建最终交易的集体检查点，以及随时间推移的 Sui 状态。轻客户端可以使用在检查点上具有法定权益的证书来有效地验证对象和发出的事件的最新状态。
A check pointing mechanism is necessary for committee reconfiguration between epochs. More frequent checkpoints are useful to light clients, and may also be used by authorities to compress their internal data structures as well as synchronize their state with other authorities more efficiently.
检查点机制对于世代之间的委员会重新配置是必要的。更频繁的检查点对轻客户端很有用，也可能被当局用来压缩其内部数据结构以及更有效地与其他当局同步他们的状态。

##4.6 Bridges

Native support for light clients and shared objects managed by Byzantine agreement allows Sui to support two-way bridges to other blockchains [13]. The trust assumption of such bridges reflect the trust assumptions of Sui and the other blockchain, and do not have to rely on trusted oracles or hardware if the other blockchain also supports light clients [7].
对拜占庭协议管理的轻客户端和共享对象的原生支持允许 Sui 支持到其他区块链的双向桥接 [13]。这种桥梁的信任假设反映了 Sui 和其他区块链的信任假设，如果其他区块链也支持轻客户端 [7]，则不必依赖可信的预言机或硬件。

Bridges are used to import an asset issued on another blockchain, to represent it and use it as a wrapped asset within the Sui system. Eventually, the wrapped asset can be unlocked and transferred back to a user on the native blockchain.
桥用于导入在另一个区块链上发行的资产，以表示它并将其用作 Sui 系统中的包装资产。最终，包裹的资产可以被解锁并转移回本地区块链上的用户。
Bridges can also allow assets issued on Sui to be locked, and used as wrapped assets on other blockchains. Eventually, the wrapped object on the other system can be destroyed, and the object on Sui updated to reflect any changes of state or ownership, and unlocked.
Bridges 还可以让在 Sui 上发行的资产被锁定，并用作其他区块链上的包装资产。最终，可以销毁另一个系统上的包装对象，并更新 Sui 上的对象以反映状态或所有权的任何更改，并解锁。

The semantics of bridged assets are of some importance to ensure wrapped assets are useful. Fungible assets bridged across blockchains can provide a richer wrapped representation that allows them to be divisible and transferable when wrapped.
桥接资产的语义对于确保包装资产有用具有一定的重要性。跨区块链桥接的可替代资产可以提供更丰富的包装表示，使它们在包装时可以分割和转移。
Nonfungible assets are not divisible, but only transferable. They may also support other operations that mutates their state in a controlled manner when wrapped, which may necessitate custom smart contract logic to be executed when they are bridged back and unwrapped.
不可替代资产不可分割，只能转让。它们还可能支持其他操作，这些操作在包装时以受控方式改变其状态，这可能需要在桥接和解包时执行自定义智能合约逻辑。
Sui is flexible and allows smart contract authors to define such experiences, since bridges are just smart contracts implemented in Move rather than native Sui concepts – and therefore can be extended using the composability and safety guarantees Move provides.
Sui 非常灵活，允许智能合约作者定义此类体验，因为桥接只是在 Move 中实现的智能合约，而不是原生的 Sui 概念——因此可以使用 Move 提供的可组合性和安全保证进行扩展。

##4.7 Committee Reconfiguration(委员会重组)

Reconfiguration occurs between epochs when a committee 𝐶𝑒 is replaced by a committee 𝐶𝑒′, where 𝑒 ′ = 𝑒 + 1. Reconfiguration safety ensures that if a transaction Tx was committed at 𝑒 or before, no conflicting transaction can be committed after 𝑒. Liveness ensures that if Tx was committed at or before 𝑒, then it must also be committed after 𝑒.
当委员会𝐶𝑒被委员会𝐶𝑒'取代时，重新配置发生在时期之间，其中𝑒'=𝑒+1。重新配置安全性确保如果交易Tx在𝑒或之前提交，则在𝑒之后不会提交任何冲突事务。 Liveness 确保如果 Tx 在 𝑒 或之前提交，那么它也必须在 𝑒 之后提交。

We leverage the Sui smart contract system to perform a lot of the work necessary for reconfiguration. Within Sui a system smart contract allows users to lock and delegate stake to candidate authorities. During an epoch, owners of coins are free to delegate by locking tokens, undelegate by unlocking tokens or change their delegation to one or more authorities.
我们利用 Sui 智能合约系统来执行大量重新配置所需的工作。在 Sui 中，系统智能合约允许用户锁定股权并将其委托给候选机构。在一个世代内，代币所有者可以通过锁定代币自由委托，通过解锁代币取消委托或将他们的委托更改为一个或多个授权机构。

Once a quorum of stake for epoch 𝑒 vote to end the epoch, authorities exchange information to commit to a checkpoint, determine the next committee, and change the epoch. First, authorities run a check pointing protocol, with the help of an agreement protocol [9], to agree on a certified checkpoint for the end of epoch 𝑒. The checkpoint contains the union of all transactions, and potentially resulting objects, that have been processed by a quorum of authorities.
一旦世代的法定人数 𝑒 投票结束纪元，权威机构交换信息以提交检查点，确定下一个委员会，并更改世代。首先，权威机构在协议 [9] 的帮助下运行检查点协议，以就世代 𝑒 结束时的认证检查点达成一致。检查点包含所有交易的联合，以及可能产生的对象，这些交易已由法定机构处理。
As a result if a transaction has been processed by a quorum of authorities, then at least one honest authorities that processed it will have its processed transactions included in the end-of-epoch checkpoint, ensuring the transaction and its effects are durable across epochs. Furthermore, such a certified checkpoint guarantees that all transactions are available to honest authorities of epoch 𝑒.
因此，如果一笔交易已由法定人数的权威机构处理，那么至少有一个处理它的诚实权威机构会将其处理过的交易包含在世代末检查点中，以确保交易及其影响跨世代持久。此外，这样一个经过认证的检查点保证所有交易都可以提供给纪元𝑒的诚实权威机构。

The stake delegation at the end-of-epoch checkpoint is then used to determine the new set of authorities for epoch 𝑒 + 1. Both a quorum of the old authorities stake and a quorum of the new authority stake signs the new committee 𝐶𝑒′, and checkpoint at which the new epoch commences. Once both set of signatures are available the new set of authorities start processing transactions for the new epoch, and old authorities may delete their epoch signing keys.
然后使用世代末检查点的权益委托来确定世代 𝑒 + 1 的新授权集。旧授权权益的法定人数和新授权权益的法定人数都签署了新委员会𝐶𝑒'，和新世代开始的检查点。一旦两组签名都可用，新的授权机构就开始处理新世代的交易，而旧的授权机构可以删除他们的世代签名密钥。

Recovery. It is possible due to client error or client equivocation for an owned object to become ‘locked’ within an epoch, preventing any transaction concerning it from being certified (or finalized).
恢复。由于客户端错误或客户端模棱两可，拥有的对象可能会在一个时期内“锁定”，从而阻止与它相关的任何交易被认证（或最终确定）
For example, a client signing two different transactions using the same owned object version, with half of authorities signing each, would be unable to form a certificate requiring a quorum of signatures on any of the two certificates. Recovery ensures that once epochs change such objects are again in a state that allows them to be used in transactions.
例如，客户端使用相同的拥有对象版本签署两个不同的交易，每个授权机构签署一半，将无法形成需要在两个证书中的任何一个上进行法定人数签名的证书。恢复确保一旦世代改变，这些对象再次处于允许它们在交易中使用的状态。
Since, no certificate can be formed, the original object is available at the start of the next epoch to be operated on. Since transactions contain an epoch number, the old equivocating transactions will not lock the object again, giving its owner a chance to use it.
由于无法形成证书，原始对象在下一个要操作的纪元开始时可用。由于交易包含世代号，旧的模棱两可的交易不会再次锁定该对象，从而为其所有者提供使用它的机会。

Rewards & cryptoeconomics. Sui has a native token SUI, with a fixed supply. SUI is used to pay for gas, and is also be used as delegated stake on authorities within an epoch.
奖励和加密经济学。 Sui 有一个原生代币 SUI，供应量固定。 SUI 用于支付 gas 费用，也被用作一个 epoch 内授权的委托权益。
The voting power of authorities within this epoch is a function of this delegated stake. At the end of the epoch fees collected through all transactions processed are distributed to authorities according to their contribution to the operation of Sui, and in turn they share some of the fees as rewards to addresses that delegated stake to them. We postpone a full description of the token economics of Sui to a dedicated paper.
这个时期内权威机构的投票权是这个委托权益的函数。在 epoch 结束时，通过处理的所有交易收取的费用将根据他们对 Sui 运营的贡献分配给权威机构，然后他们将部分费用作为奖励分享给委托给他们的地址。我们将对 Sui 代币经济学的完整描述推迟到专门的论文中。

##4.8 Authority & Replica Updating(权限和副本更新)

Client-driven. Due to client failures or non-byzantine authority failures, some authorities may not have processed all certificates. As a result causally related transactions depending on missing objects generated by these certificates would be rejected.
客户驱动。由于客户端故障或非拜占庭权限故障，某些权限可能没有处理所有证书。因此，依赖于由这些证书生成的缺失对象的因果关系交易将被拒绝。
However, a client can always update an honest authority to the point where it is able to process a correct transaction. It may do this using its own store of past certificates, or using one or more other honest authorities as a source for past certificates.
但是，客户端始终可以将诚实授权更新到能够处理正确交易的程度。它可以使用自己的过去证书存储库或使用一个或多个其他诚实的权威机构作为过去证书的来源来执行此操作。

Given a certificate 𝑐 and a 𝐶𝑡𝑣 store that includes𝑐 and its causal history, a client can update an honest authority 𝑣′ to the point where 𝑐 would also be applied. This involves finding the smallest set of certificates not in 𝑣′ such that when applied the Objects in 𝑣′ include all inputs of 𝑐. Updating a lagging authority 𝐵 using a store 𝐶𝑡𝑣 including the certificate TxCert involves:
给定证书𝑐 和包含𝑐 及其因果历史的𝐶𝑡𝑣 存储，客户端可以将诚实授权𝑣′ 更新到也将应用𝑐 的点。这涉及找到不在 𝑣′ 中的最小证书集，以便在应用 𝑣′ 中的对象时包括 𝑐 的所有输入。使用商店 𝐶𝑡𝑣 更新滞后权威 𝐵 包括证书 TxCert 涉及：

• The client maintains a list of certificates to sync, initially set to contain just TxCert.
• 客户端维护要同步的证书列表，最初设置为只包含TxCert。

• The client considers the last TxCert requiring sync. It extracts the Tx within the TxCert and derives all its input objects (using Input(Tx)).
• 客户端考虑需要同步的最后一个TxCert。它提取 TxCert 中的 Tx 并派生其所有输入对象（使用 Input(Tx)）。

• For each input object it checks whether the Tx that generated or mutated last (using the Sync𝑣 index on 𝐶𝑡𝑣 ) has a certificate within 𝐵, otherwise its certificate is read from 𝐶𝑡𝑣 and added to the list of certificates to sync.
• 对于每个输入对象，它检查最后生成或变异的Tx（使用𝐶𝑡𝑣 上的Sync𝑣 索引）是否在𝐵 内有一个证书，否则它的证书从𝐶𝑡𝑣 中读取并添加到要同步的证书列表中。

• If no more certificates can be added to the list (because no more inputs are missing from 𝐵) the certificate list is sorted in a causal order and submitted to 𝐵.
• 如果没有更多的证书可以添加到列表中（因为𝐵 中没有更多的输入丢失），则证书列表按因果顺序排序并提交给𝐵。

The algorithm above also applies to updating an object to a specific version to enable a new transaction. In this case the certificate for the Tx that generated the object version, found in Sync𝑣 [ObjRef], is submitted to the lagging authority. Once it is executed on 𝐵 the object at the correct version will become available to use.
上述算法也适用于将对象更新到特定版本以启用新交易。在这种情况下，生成对象版本的 Tx 的证书，在 Sync𝑣 [ObjRef] 中找到，被提交给滞后的授权机构。一旦它在 𝐵 上执行，正确版本的对象将可供使用。

A client performing this operation is called a relayer. There can be multiple relayers operating independently and concurrently. They are untrusted in terms of integrity, and their operation is keyless. Besides clients, authorities can run the relayer logic to update each other, and replicas operating services can also act as relayers to update lagging authorities.
执行此操作的客户端称为中继器。可以有多个独立和同时运行的中继器。他们在完整性方面不受信任，并且他们的操作是无钥匙的。除了客户端之外，权威机构还可以运行中继器逻辑来相互更新，副本操作服务也可以充当中继器来更新滞后的权威机构。

Bulk. Authorities provide facilities for a follower to receive updates when they process a certificate. This allows replicas to maintain an up-to-date view of an authorty’s state. Furthermore, authorities may use a push-pull gossip network to update each other of the latest processed transaction in the short term and to reduce the need for relayers to perform this function.
大部分。权威机构为追随者提供便利，以便他们在处理证书时接收更新。这允许副本维护权威状态的最新视图。此外，权威机构可以使用推拉gossip网络在短期内相互更新最新处理的交易，并减少中继器执行此功能。
In the longer term lagging authorities may use periodic state commitments, at epoch boundaries or more frequently, to ensure they have processed a complete set of certificates up to certain check points.
从长远来看，落后的权威机构可能会在世代边界或更频繁地使用周期性的状态承诺，以确保他们已经处理了一整套直到某些检查点的证书。

#5 SCALING AND LATENCY(缩放和延迟)

The Sui system allows scaling though authorities devoting more resources, namely CPUs, memory, network and storage within a machine or over multiple machines, to the processing of transactions.
Sui 系统允许权威机构通过投入更多资源（即一台机器或多台机器上的 CPU、内存、网络和存储）来扩展交易处理。
More resources lead to an increased ability to process transactions, leading to increased fees income to fund these resources.
更多的资源导致处理交易的能力增强，从而导致为这些资源提供资金的费用收入增加。
More resources also results in lower latency, as operations are performed without waiting for necessary resources to become available.
更多的资源也会导致更低的延迟，因为无需等待必要的资源可用即可执行操作。

Throughput. To ensure that more resources result in increased capacity quasi-linearly, the Sui design aggressively reduces bottlenecks and points of synchronization requiring global locks within authorities.
吞吐量。为了确保更多的资源导致容量准线性增加，Sui 设计积极减少了瓶颈和需要在权限内进行全局锁定的同步点。
Processing transactions is cleanly separated into two phases, namely (1) ensuring the transaction has exclusive access to the owned or shared objects at a specific version, and (2) then subsequently executing the transaction and committing its effects.
处理事务清楚地分为两个阶段，即 (1) 确保事务具有对特定版本的拥有或共享对象的独占访问权，以及 (2) 随后执行交易并提交其效果。

Phase (1) requires a transaction acquiring distributed locks at the granularity of objects. For owned objects this is performed trough a reliable broadcast primitive, that requires no global synchronization within the authority, and therefore can be scaled through sharding the management of locks across multiple machines by ObjID. For transactions involving shared objects sequencing is required using a consensus protocol, which does impose a global order on these transactions and has the potential to be a bottleneck.
阶段（1）需要交易获取对象粒度的分布式锁。对于拥有的对象，这是通过可靠的广播原语执行的，它不需要权限内的全局同步，因此可以通过 ObjID 跨多台机器对锁的管理进行分片来扩展。对于涉及共享对象的事务，需要使用共识协议进行排序，这确实对这些事务施加了全局顺序，并有可能成为瓶颈。
However, recent advances on engineering high-throughput consensus protocols [9] demonstrate that sequential execution is the bottleneck in state machine replication, not sequencing. In Sui, sequencing is only used to determine a version for the input shared object, namely incrementing an object version number and associating it with the transaction digest, rather than performing sequential execution.
然而，工程高吞吐量共识协议的最新进展 [9] 表明顺序执行是状态机复制的瓶颈，而不是排序。在Sui中，排序仅用于确定输入共享对象的版本，即递增对象版本号并将其与交易摘要相关联，而不是执行顺序执行。

Phase (2) takes place when the version of all input objects is known to an authority (and safely agreed across authorities) and involves execution of the Move transaction and commitment of its effects. Once the version of input objects is known, execution can take place completely in parallel. Move virtual machines on multiple cores or physical machines read the versioned input objects, execute, and write the resulting objects from and to stores.
阶段 (2) 发生在所有输入对象的版本为权威机构所知（并在权威机构之间安全地达成一致）并且涉及 Move 交易的执行及其效果的承诺。一旦知道输入对象的版本，就可以完全并行地执行。在多核或物理机上移动虚拟机读取版本化的输入对象，执行并将结果对象写入存储。
The consistency requirements on stores for objects and transactions(besides the order lock map) are very loose, allowing scalable distributed key-value stores to be used internally by each authority. Execution is idempotent, making even crashes or hardware failures on components handling execution easy to recover from.
对象和交易存储的一致性要求（除了顺序锁映射）非常宽松，允许每个机构在内部使用可扩展的分布式键值存储。执行是幂等的，即使处理执行的组件发生崩溃或硬件故障也很容易恢复。

As a result, execution for transactions that are not causally related to each other can proceed in parallel. Smart contract designers may therefore design the data model of objects and operations within their contracts to take advantage of this parallelism.
因此，相互之间没有因果关系的交易的执行可以并行进行。因此，智能合约设计者可以在他们的合约中设计对象和操作的数据模型，以利用这种并行性。

Check-pointing and state commitments are computed off the critical transaction processing path to not block the handling of fresh transactions. These involve read operations on committed data rather than requiring computation and agreement before a transaction reaches finality. Therefore they do not affect the latency or throughput of processing new transactions, and can themselves be distributed across available resources.
检查点和状态承诺是在关键交易处理路径之外计算的，不会阻止新交易的处理。这些涉及对已提交数据的读取操作，而不是在交易达到最终结果之前需要计算和协议。因此，它们不会影响处理新交易的延迟或吞吐量，并且它们本身可以分布在可用资源中。

Reads can benefit from very aggressive, and scalable caching. Authorities sign and make available all data that light clients require for reads, which may be served by distributed stores as static data.
读取可以从非常积极且可扩展的缓存中受益。权威机构签署并提供轻客户端读取所需的所有数据，这些数据可能由分布式存储作为静态数据提供。
Certificates act as roots of trust for their full causal history of transactions and objects. State commitments further allow for the whole system to have regular global roots of trust for all state and transactions processed, at least every epoch or more frequently.
证书作为交易和对象完整因果历史的信任根。状态承诺进一步允许整个系统对所有状态和处理的交易具有定期的全球信任根，至少每个时期或更频繁。

Latency. Smart contract designers are given the flexibility to control the latency of operations they define, depending on whether they involve owned or shared objects. Owned objects rely on a reliable broadcast before execution and commit, which requires two round trips to a quorum of authorities to reach finality.
潜伏。智能合约设计者可以灵活地控制他们定义的操作的延迟，具体取决于它们是涉及拥有的还是共享的对象。拥有的对象在执行和提交之前依赖于可靠的广播，这需要两次往返到授权机构的法定人数才能达到最终结果。
Operations involving shared objects, on the other hand, require a consistent broadcast to create a certificate, and then be processed within a consensus protocol, leading to increased latency (4 to 8 round trips to quorums as of [9]).
另一方面，涉及共享对象的操作需要一致的广播来创建证书，然后在共识协议中进行处理，从而导致延迟增加（从 [9] 开始需要 4 到 8 次往返仲裁）。



















