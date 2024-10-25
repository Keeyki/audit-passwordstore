### [H-1] Storing the password on-chain makes it visable to anyone, and no longer private (Root Cause + Impact)

**Description:**
All data on the blockchain is visible to anyone and can be directly read from the blockchain. The `PasswordStore::s_password` variable is intended to be a private variable and should only be accessed through the `PasswordStore::getPassword` function, which is intended to be called solely by the contract owner.
Below, we demonstrate a method to read any on-chain data.

**Impact:**
Everyone can read the private password, severly breaking the functionality of the protocol.

**Proof of Concept:**
The below test case shows how anyone can read the password directly from the blockchain.

1.Create a locally running chain

```bash
make anvil
```

2. Deploy the contract to the chain

```
make deploy
```

3. Run the storage tool
   we use `1` because that's the storage slot of `s_password` in the contract.

```
cast storage <ADDRESS_HERE> 1 --rpc-url http://127.0.0.1:8545
```

You'll get an output that looks like this:
`0x6d7950617373776f726400000000000000000000000000000000000000000014`

You can then parse that hex to a string with:

```
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014

```

And get an output of :
myPsassword

**Recommended Mitigation:**
As a result, the contract's overall architecture may need to be reconsidered. One approach could be to encrypt the password off-chain and store only the encrypted version on-chain. This would require the user to remember an additional off-chain password for decryption. Additionally, it may be wise to remove the view function to avoid the risk of users inadvertently sending a transaction containing this decryption key.

### [H-2] `PasswordStore::setPassword` has no access controls, meaning a non-owner could change the password

**Description:**
The `PasswordStore::setPassword` function is set to be an `external` function, however the purpose of the smart contract and function's natspec indicate that `This function allows only the owner to set a new password.`

**Impact:**
Anyone can set/change the stored password, severly breaking the contract's intended functionality

**Proof of Concept:** add the following to the `PasswordStore.t.sol` test file.

<details>

    function test_anyone_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "mynewpassword";
        passwordStore.setPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }

</details>

**Recommended Mitigation:**
Add an access control conditionnal to the `setPassword` function.

```
if(msg.sender != s_owner) {
    revert PasswordStore_NotOwner();
}
```

**Title:** [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that doesn't exist, causing the natspec to be incorrect.

**Description:**
'''

/\*

- @notice This allows only the owner to retrieve the password.
- @param newPassword The new password to set.
  \*/
  function getPassword() external view returns (string memory) {}

The `PasswordStore::getPassword` function signature is `getPassword()` while the natspec says it should be `getPassword(string)`.

**Impact:**
The natspec is incorrect
**Proof of Concept:**

**Recommended Mitigation:** Remove the incorrect natspec line

```diff
-    * @param newPassword The new password to set.
```
