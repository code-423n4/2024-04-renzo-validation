[L-0] Oracle price updates can be frontrunned.

A user can front run the price updates of the oracle and get the most of out of the transaction. 

[L-1] L1 data fees are not reimbursed

Some L2s needs to transfer data to L1 through calldata. This fees is not taken into account when gas refund is paid. 

The similar issue can be found here: 

https://github.com/sherlock-audit/2024-04-xkeeper-judging/issues/57

The recommendation given on the issue above is suitable enough to mitigate this.