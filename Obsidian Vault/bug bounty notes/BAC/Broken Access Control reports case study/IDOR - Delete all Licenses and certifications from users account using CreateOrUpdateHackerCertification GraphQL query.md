[harshdranjan](https://hackerone.com/harshdranjan)

 submitted a report to [**HackerOne**](https://hackerone.com/security).

August 24, 2023, 3:52pm UTC

**Summary:** Hey team,

While editing our **Licenses and certifications** if we change the ID number we can delete other users **Licenses and certifications**. it simply can be done by editing the ID number in our graphql query. If change the ID from 1 to X possible range then we can delete all the **Licenses and certifications** present between these.

### Steps To Reproduce

1. Log in to your own account in two browsers A and B with User A and User B
2. Create your own *_Licenses and certifications_ in both the account
3. Now edit your own *_Licenses and certifications_ and Intercept this using a Burp Proxy
4. Now In the body change the **ID** number and you will be able to delete all the **Licenses and certifications** present in HackerOne
5. For now change the ID to the **Licenses and certifications** ID of the Other account and it will be deleted.

PoC Video: ████

## Impact

Able to delete all the **Licenses and certifications** present in HackerOne