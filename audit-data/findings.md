<!-- (1)Convince the protocol this is an issue
(2) How bad this issue is 
(3) How to fix the issue -->



<!-- ### [S-#] TITLE (Root Cause + Impact) -->
<!-- ### [S-#] Variables stored in storage on-chain are visible to anyone , no matter the solidity visibility keyword meaning the password is actually not a private pasword -->

### [S-#]

<!-- Shorter form -->
### [H-1] Storing the password on-chain makess it visible to anyoone and, no longer private

**Description:** All data stored onChain is visible to anyone , and can be read directly from the blockchain. The `PasswordStore::s_passsword` variable is intended to be private variable and only accessed through the `PasswordStore::getPassword` function, which is intended to be only called by the owner of the contract

We show one such method of reading any data off chain below.

**Impact:**  Anyone can read the private password, severly breaking the functionality of the protocol.

**Proof of Concept:**(proof of code)

The below test case shows how anyone can read the password directtly from the blockchain

1. Create a locally running chain
```bash
make anvil
```

2.Deploy the contract to the chain
```
make  deploy
```

3.Run the storage tool
```
we use `1` because that's the storage slot of `s_password` in the contract.
```
cast storage <ADDRESS_HERE>  1 --rpc-url http://127.0.0.1:8545
```

You'll get an output that looks like this:
`0x6d7950617373776f726400000000000000000000000000000000000000000014`

You can then parse that hex to a string with'

```
cast parse-bytes32-string 0x6d7950617373776f726400000000000000000000000000000000000000000014
```

And get an output of;

"myPasssword"
```





**Recommended Mitigation:**  Due to this, the overrall architecture of the contract should rethought. One could encrypt the passsword off-chain, and then store the encrypted password on-chain. This world require a user to rememmber another password off-chain to  decrypt the password. However, you'll also likely want to remove the view function as you would't want the user to accidentally send a transaction with the password that decrypts your password.

    ## Likelyhood and Impact
        -Impact:HIGH
        -Likelyhood: High
        -Severity:HIGH

<!-- ## HIGH
-Worst offenders -> Least bad
## Medium
## Low -->


### [H-2] `PasswordStore::setPassword` has no access control, meaning a non-owner could change the password.

**Description:** The `PasswordStore::setPassword` function is set to be an `external` function, however, the natspect of the function overrall purpose of the smart contract is that `This function allows only the owner to set a new password.`

```javascript
   function setPassword(string memory newPassword) external {
   ==> //@audit - There are no access control
        s_password = newPassword;
        emit SetNetPassword();
    }
```

**Impact:** Anyone can set/change the password of the contract, severly breaking the contract functionalities.

```javascript
 function setPassword(string memory newPassword) external {
        s_password = newPassword;
        emit SetNetPassword();
    }

```

**Proof of Concept:** Add the following to the `passwordStore.t.sol` test file.
<details>
    <summary>Code</summary>
    ```javascript

        function test_anyone_can_get_password(address randomAddress) public {
            //to be sure that the caller iss not the owner
            vm.assume(randomAddress != owner);

            vm.prank(randomAddress);
            string memory expectedPassword = "myNewPassword";
            passwordStore.setPassword(expectedPassword);

            vm.prank(owner);
            string memory actualPassword = passwordStore.getPassword();

            //comparing if the actual password is the same ass the  expected one
            assertEq(actualPassword, expectedPassword);
        }
    ```
</details>



**Recommended Mitigation:**  Add an access control conditional to the `setPassword` function.

```javasscript
   if(msg.ssender!=s_owner){
    revert PasswordStore__NotOwner();
   }
```

    ## Likelyhood and Impact
    -Impact:HIGH
    -Likelyhood: High
    -Severity:HIGH


### [I-1] The `PasswordStore::getPassword` napspec indicates a parameter that doesn't exist, causing the napspec to be incorrect.

    **Description:** 
    ```javascript
        /*
        * @notice This allows only the owner to retrieve the password.
        * @audit There is no newPassword parameter!
        * @param newPassword The new password to set.
        */
        function getPassword() external view returns (string memory) {

    ```
    The `passwordStore::getPassword` function ssignature is `getPassword()` which the natspec say it should be `getPassword(string)`

    **Impact:**  The napspec is incorrect


    **Recommended Mitigation:** Remove the natspec line.

    ```diff

    - * @param newPassword The new password to set.
    ```



## Likelyhood and Impact
-Impact:NONE
-Likelyhood: HIGH
-Severity:Informational/Gas/Non-crits

Informational: Hey, this isn't a bug, but you should know ...



