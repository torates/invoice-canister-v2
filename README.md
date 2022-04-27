# Invoice Canister Implementation
This repo provides clear instructions on how to implement Kyle Peacock's original Invoice Canister, and how to use it for local development.


## How it works
![image](https://user-images.githubusercontent.com/59488791/163595058-837926e1-b174-49ce-8a3a-73e4658cbb73.png)
*Example of the userflow in Overchute's Invoice implementation*

The Invoice Canister, at it's core, takes a two step approach to processing payments, the first one just creating an object will all the necessary information and the second one just checking that the ICP has been received, thus as such it contains two main functions representing both of these steps, the `create_invoice()`, which returns a type *CreateInvoiceResult* and the `verify_invoice()` function, which returns a type *VerifyInvoiceResult*.
It's in between both steps that the user is requested to sign the transaction. 

PD. Keep in mind this is NOT a fixed design. The invoice canister can be changed at will to serve the desired purposes of a project - *that is the beauty of coding*

#### Step 0: Initiating payment flow from the frontend
The first thing is initiating the payment flow. This can be achieved with an input/form in the frontend.

For example, in Overchute's CrowsdsaleContribution form, the user is prompted to type an amount of ICP to contribute, this is then sent to the `create_invoice()` alongside all the other desired information (for example, the crowdsale that it's contributing to), and added to the created `Invoice`.

#### Step 1: The `create_invoice()` function
As shown before, the `create_invoice()` function can be made to accept arguments, for them to be added to the returned `Invoice` type if process is successful.

The returned invoice has a `destination` account ID, that by default is generated using the principal of the invoice canister plus using the caller and the Invoice `id` for the subaccount.

#### Step 2: Asking for payment
After the `Invoice` is created, the next step is asking for a transfer to the destination account.

For example, with Plug wallet it would look something similar to:

```

  let invoice = await actor.create_invoice(crodwsaleId, contribution);

  let destinationInvoice = invoice.ok['invoice']
  let destinationAcc = destinationInvoice.destination['text']

  const newcontribution = contribution * 100_000_000
  const requestTransferArg = {
    to: destinationAcc, //transfer to the destination account
    amount: newcontribution,
  };
  const transfer = await window.ic?.plug?.requestTransfer(requestTransferArg);

```
This is creating an invoice for the payment, and then sending a ```contribution``` amount of ICP to the generated account ID.

#### Step 3: Verifying the invoice

The ```verify_invoice()```

## For Local Development

## Types
```
public type Token = {
    symbol : Text;
  };
  public type TokenVerbose = {
    symbol : Text;
    decimals : Int;
    meta : ?{
      Issuer : Text;
    };
  };
  public type AccountIdentifier = {
    #text : Text;
    #principal : Principal;
    #blob : Blob;
  };
  public type Details = {
    description : Text;
    meta : Blob;
  };
  public type Permissions = {
      canGet : [Principal];
      canVerify : [Principal];
  };
  public type Invoice = {
    id : Nat;
    creator : Principal;
    details : ?Details;
    permissions : ?Permissions;
    amount : Nat;
    amountPaid : Nat;
    token : TokenVerbose;
    verifiedAtTime : ?Time.Time;
    paid : Bool;
    destination : AccountIdentifier;
  };
// #endregion

/**
* Service Args and Result Types
*/

// #region create_invoice
  public type CreateInvoiceArgs = {
    amount : Nat;
    token : Token;
    permissions: ?Permissions;
    details : ?Details;
  };
  public type CreateInvoiceResult = Result.Result<CreateInvoiceSuccess, CreateInvoiceErr>;
  public type CreateInvoiceSuccess = {
    invoice : Invoice;
  };
  public type CreateInvoiceErr = {
    message : ?Text;
    kind : {
      #BadSize;
      #InvalidToken;
      #InvalidAmount;
      #InvalidDestination;
      #InvalidDetails;
      #MaxInvoicesReached;
      #Other;
    };
  };
// #endregion

// #region Get Destination Account Identifier
  public type GetDestinationAccountIdentifierArgs = {
    token : Token;
    caller : Principal;
    invoiceId : Nat;
  };
  public type GetDestinationAccountIdentifierResult = Result.Result<GetDestinationAccountIdentifierSuccess, GetDestinationAccountIdentifierErr>;
  public type GetDestinationAccountIdentifierSuccess = {
    accountIdentifier : AccountIdentifier;
  };
  public type GetDestinationAccountIdentifierErr = {
    message : ?Text;
    kind : {
        #InvalidToken;
        #InvalidInvoiceId;
        #Other;
    };
  };
// #endregion

// #region get_invoice
  public type GetInvoiceArgs = {
    id : Nat;
  };
  public type GetInvoiceResult = Result.Result<GetInvoiceSuccess, GetInvoiceErr>;
  public type GetInvoiceSuccess = {
    invoice : Invoice;
  };
  public type GetInvoiceErr = {
    message : ?Text;
    kind : {
      #InvalidInvoiceId;
      #NotFound;
      #NotAuthorized;
      #Other;
    };
  };
// #endregion

// #region get_balance
  public type GetBalanceArgs = {
    token : Token;
  };
  public type GetBalanceResult = Result.Result<GetBalanceSuccess, GetBalanceErr>;
  public type GetBalanceSuccess = {
    balance : Nat;
  };
  public type GetBalanceErr = {
    message : ?Text;
    kind : {
      #InvalidToken;
      #NotFound;
      #Other;
    };
  };
// #endregion

// #region verify_invoice
  public type VerifyInvoiceArgs = {
    id : Nat;
  };
  public type VerifyInvoiceResult = Result.Result<VerifyInvoiceSuccess, VerifyInvoiceErr>;
  public type VerifyInvoiceSuccess = {
    #Paid : {
      invoice : Invoice;
    };
    #AlreadyVerified : {
      invoice : Invoice;
    };
  };
  type VerifyInvoiceErr = {
    message : ?Text;
    kind : {
      #InvalidInvoiceId;
      #NotFound;
      #NotYetPaid;
      #NotAuthorized;
      #Expired;
      #TransferError;
      #InvalidToken;
      #InvalidAccount;
      #Other;
    };
  };
// #endregion

// #region transfer
  public type TransferArgs = {
    amount : Nat;
    token : Token;
    destination : AccountIdentifier;
  };
  public type TransferResult = Result.Result<TransferSuccess, TransferError>;
  public type TransferSuccess = {
    blockHeight : Nat64;
  };
  public type TransferError = {
    message : ?Text;
    kind : {
      #BadFee;
      #InsufficientFunds;
      #InvalidToken;
      #InvalidDestination;
      #Other;
    };
  };
// #endregion

// #region get_caller_identifier
  public type GetAccountIdentifierArgs = {
    token : Token;
    principal : Principal;
  };
  public type GetAccountIdentifierResult = Result.Result<GetAccountIdentifierSuccess, GetAccountIdentifierErr>;
  public type GetAccountIdentifierSuccess = {
    accountIdentifier : AccountIdentifier;
  };
  public type GetAccountIdentifierErr = {
    message : ?Text;
    kind : {
      #InvalidToken;
      #Other;
    };
  };
// #endregion

// #region accountIdentifierToBlob
  public type AccountIdentifierToBlobArgs = {
    accountIdentifier : AccountIdentifier;
    canisterId : ?Principal;
  };
  public type AccountIdentifierToBlobResult = Result.Result<AccountIdentifierToBlobSuccess, AccountIdentifierToBlobErr>;
  public type AccountIdentifierToBlobSuccess = Blob;
  public type AccountIdentifierToBlobErr = {
    message : ?Text;
    kind : {
      #InvalidAccountIdentifier;
      #Other;
    };
  };
// #endregion

// #region accountIdentifierToText
  public type AccountIdentifierToTextArgs = {
    accountIdentifier : AccountIdentifier;
    canisterId : ?Principal;
  };
  public type AccountIdentifierToTextResult = Result.Result<AccountIdentifierToTextSuccess, AccountIdentifierToTextErr>;
  public type AccountIdentifierToTextSuccess = Text;
  public type AccountIdentifierToTextErr = {
    message : ?Text;
    kind : {
      #InvalidAccountIdentifier;
      #Other;
    };
  };
// #endregion

// #region ICP Transfer
  public type Memo = Nat64;
  public type SubAccount = Blob;
  public type TimeStamp = {
    timestamp_nanos : Nat64;
  };
  public type ICPTokens = {
    e8s : Nat64;
  };
  public type ICPTransferError = {
    message : ?Text;
    kind : {
      #BadFee : {
        expected_fee : ICPTokens;
      };
      #InsufficientFunds : {
        balance : ICPTokens;
      };
      #TxTooOld : {
        allowed_window_nanos : Nat64;
      };
      #TxCreatedInFuture;
      #TxDuplicate : {
        duplicate_of : Nat;
      };
      #Other;
    }
  };

  public type ICPTransferArgs = {
    memo : Memo;
    amount : ICPTokens;
    fee : ICPTokens;
    from_subaccount : ?SubAccount;
    to : AccountIdentifier;
    created_at_time : ?TimeStamp;
  };

  public type ICPTransferResult = Result.Result<TransferSuccess, ICPTransferError>;
  ```
