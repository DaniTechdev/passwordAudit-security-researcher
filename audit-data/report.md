---
title: Protocol Audit Report
author: Nneji Daniel T
date: March 7, 2023
header-includes:
  - \usepackage{titling}
  - \usepackage{graphicx}
---

\begin{titlepage}
    \centering
    \begin{figure}[h]
        \centering
        \includegraphics[width=0.5\textwidth]{logo.pdf} 
    \end{figure}
    \vspace*{2cm}
    {\Huge\bfseries Protocol Audit Report\par}
    \vspace{1cm}
    {\Large Version 1.0\par}
    \vspace{2cm}
    {\Large\itshape NnejiDaniel\par}
    \vfill
    {\large \today\par}
\end{titlepage}

\maketitle

<!-- Your report starts here! -->

Prepared by: [Nneji Daniel](https://cyfrin.io)
Lead Auditors/Security Ressearcher: 
- xxxxxxx

# Something

# Table of Contents
- [Something](#something)
- [Table of Contents](#table-of-contents)
- [Protocol Summary](#protocol-summary)
- [Disclaimer](#disclaimer)
- [Risk Classification](#risk-classification)
- [Audit Details](#audit-details)
  - [Scope](#scope)
  - [Roles](#roles)
- [Executive Summary](#executive-summary)
  - [Issues found](#issues-found)
  - [Findings](#findings)
  - [High](#high)
    - [\[H-1\] Storing the password on-chain makess it visible to anyoone and, no longer private](#h-1-storing-the-password-on-chain-makess-it-visible-to-anyoone-and-no-longer-private)
    - [\[H-2\] `PasswordStore::setPassword` has no access control, meaning a non-owner could change the password.](#h-2-passwordstoresetpassword-has-no-access-control-meaning-a-non-owner-could-change-the-password)
- [Informational](#informational)
    - [\[I-1\] The `PasswordStore::getPassword` napspec indicates a parameter that doesn't exist, causing the napspec to be incorrect.](#i-1-the-passwordstoregetpassword-napspec-indicates-a-parameter-that-doesnt-exist-causing-the-napspec-to-be-incorrect)
- [Gas](#gas)

# Protocol Summary

Password is a protocol dedicated to storage and retrieval of a ujser's password. The protocol is designed to be used by a single user , and is not designed to be used by multiple users. Only the owner should be able to set and access this pasword.

# Disclaimer

The Nneji Daniel's team makes all effort to find as many vulnerabilities in the code in the given time period, but holds no responsibilities for the findings provided in this document. A security audit by the team is not an endorsement of the underlying business or product. The audit was time-boxed and the review of the code was solely on the security aspects of the Solidity implementation of the contracts.

# Risk Classification

|            |        | Impact |        |     |
| ---------- | ------ | ------ | ------ | --- |
|            |        | High   | Medium | Low |
|            | High   | H      | H/M    | M   |
| Likelihood | Medium | H/M    | M      | M/L |
|            | Low    | M      | M/L    | L   |

We use the [CodeHawks](https://docs.codehawks.com/hawks-auditors/how-to-evaluate-a-finding-severity) severity matrix to determine severity. See the documentation for more details.

# Audit Details 

**The findings desscribed in thiss document corresponds the following commit hash:**
```
 2e8f81e263b3a9d18fab4fb5c46805ffc10a9990
```


## Scope 

```
./src/
└── PasswordStore.sol
```

## Roles

-Owner: The user who can set the password and read the password.
-Outsiders:No one else should be able to set or read the passsword.

# Executive Summary
*Add some notes about how the audit  went,type of things you found, etc.*
*We spent X hours with Z auditors using Y tools, etc*

## Issues found

| Severity | Number of Issues found |
| -------- | ---------------------- |
| High     | 2                      |
| Medium   | 0                      |
| Low      | 0                      |
| Info     | 1                      |
| Total    | 3                      |


## Findings

## High

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


# Informational

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




# Gas 