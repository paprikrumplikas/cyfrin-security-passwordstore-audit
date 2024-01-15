### [H-1] Storing the pwd on-chain makes it visible to anyone, and no longer private

**Description:** All  data stored on-chain is visible to anyone, and can be read directly from the blockchain. The `PasswordStore::s_password` variable is intended to be a private variable and only accessed through the `PasswordStore::getPassword` function, which is intended to be only called by the owner of the contract.

We show one such method of reading any data off chain below.


**Impact:** Anyone can read the private password, severely breaking the functionality of the protocol.


**Proof of Concept:** (Proof of Code)

The test case below shows how anyone can read the pwd directly from the blockchain.

1. Create a locally running chain
```bash
make anvil
```
2. Deploy the contract to the chain
```
make deploy
```

3. Run the storage tool
We use `1` because that is the storage slot of the `PasswordStore::s_password` in the contract.

```
cast storage <ADDRESS HERE> 1 --rpc-url http://127.0.0.1:8545
```

You will get an output like this:
`0x6d7950617373776f726400000000000000000000000000000000000000000014`

You can then parse this hex to a string as follows:

```
cast --parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

And get an output of:
```
myPassword
```


**Recommended Mitigation:** Due to this the overall architecture of the contract should be rethought. One could encrypt the pwd off-chain, and then store the encyprted pwd on-chain. This would require the user to remember another pwd off-chain to decrypt the pwd. However, you would also likely want to remove the view function as you would not want the user to accidentally send a transaction with the pwd that decrypts your pwd.


### [H-2] `PasswordStore::setPassword` has no access control, meaning a non-owner could change the pwd.

**Description:** The `PasswordStore::setPaswword` function is set to be an `external` function, however, the natspec of the function and overall the purpose of the smart contract is that `This function allows only the owner to set a new pwd.`

```javascript
    function setPassword(string memory newPassword) external {
@>      // @audit - There are no access controls
        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** Anyone can set/change the pwd of the contract, severely breaking the contract's intended functionality.

**Proof of Concept:** Add the following to the `PasswordStore.t.sol` file:

<details>
<summary>Code</summary> 

```javascript
    function test_anyone_can_set_password(address randomAddress) public {
        vm.assume(randomAddress != owner);
        vm.prank(randomAddress);
        string memory expectedPassword = "myNewPassword";
        passwordStore.etPassword(expectedPassword);

        vm.prank(owner);
        string memory actualPassword = passwordStore.getPassword();
        assertEq(actualPassword, expectedPassword);
    }
```
</details>


**Recommended Mitigation:** Add an access control conditional to the `setPassword` function.

```javascript
if(msg.sender != owner){
    revert PasswordStore__NotOwner();
}
```

### [I-1] The `PasswordStore::getPassword` natspec indicates a parameter that does not exist, causing the natspec to be incorrect.

**Description:** 

```javascript 
     /* 
     * @notice This allows only the owner to retrieve the password.
@>   * @param newPassword The new password to set.
     */
    function getPassword() external view returns (string memory) {
```

The `PasswordStore::getPassword` function siganture is `getPassword()`, but the natspec says it should be `getPassword(string)`.

**Impact:** The natspec is incorrect.

**Proof of Concept:** -

**Recommended Mitigation:** Remove the incorrect natspec line.

```diff
-   * @param newPassword The new password to set.
```
