# Failure of SSTORE on Low-Gas Recursive CALLs

|RSKIP          |nn           |
| :------------ |:-------------|
|**Title**      |Failure of SSTORE on Log-Gas Recursive CALLs |
|**Created**    |2019 |
|**Author**     |SDL |
|**Purpose**    |Sca, Sec, Usa |
|**Layer**      |Core |
|**Complexity** |1 |
|**Status**     |Draft |

# **Abstract**

A contract being called can call back the caller, directly or with intermediate calls, creating recursion. Some contracts are are not designed to handle recursion correctly: they rely on passing 2300 gas units to the called contract with the expectation that this will prevent recursion. However is a very bad practice as it prevents improving the opcodes costs. The easiest method to change the gas costs but yet protect these old contracts is by preventing recursion or preventing SSTORE when the call transfers less than m=8000 units of gas. This RSKIP proposes forcing SSTORE to fail on those cases.

# **Motivation**

All the horrible coin thefts that have occurred in Ethereum due to this design flaw.

# **Specification**

There is a global counter LOCKCOUNTER starts as zero when a transaction is executed. When a contract calls another contract passing less than m=8000 units of gas, LOCKCOUNTER is incremented. Whenever a contract tries to execute SSTORE and the LOCKCOUNTER is non-zero, the contract generates an OOG exception. The SSTORE is not executed. 

When a CALL that incremented the LOCKCOUNTER returns, the LOCKCOUNTER is decremented. 

# **Discussion**

There is another variant that was discarded: 

Whenever a contract calls with CALL (not DELEGATECALL nor CALLCODE) a contract that exists in the LOCKED set, it automatically returns 0 (failure). The cost of the CALL (700) is charged, but no coins are transferred (it would generate OOG) and no execution gas consumption is charged. The effect is similar as if the call had hit the maximum stack depth limit.   

This variant is more complex, requires more memory and does not represent exactly how the system would have behaved before this change. For example, a contract may call itself to query some information, even passing less than 8000 gas.


# **Copyright**

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).