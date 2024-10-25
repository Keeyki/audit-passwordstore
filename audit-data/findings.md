### [S-#] Storing the password on-chain makes it visable to anyone, and no longer private (Root Cause + Impact)

**Description:**
All data on the blockchain is visible to anyone and can be directly read from the blockchain. The `PasswordStore::s_password` variable is intended to be a private variable and should only be accessed through the `PasswordStore::getPassword` function, which is intended to be called solely by the contract owner.
Below, we demonstrate a method to read any on-chain data.

**Impact:**

**Proof of Concept:**

**Recommended Mitigation:**
