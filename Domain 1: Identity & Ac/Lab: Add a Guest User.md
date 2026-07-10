# Lab: Add a Guest User (B2B Collaboration) - AZ-500 Domain 1

## Prerequisites 
- Role: Teams administrator (UA) ✅
- External email used: shanthiraj519@gmail.com ✅

## Steps Completed
1. Signed in to Microsoft Entra admin center (https://entra.microsoft.com) as User Administrator
2. Browsed to **Entra ID** > **Users**
3. Selected **Invite external user**
4. Under **Basics**:
   - Email: shanthiraj519@gmail.com
   - Display name: (as entered)
   - Checked **Send invite message**
5. Selected **Review and invite** > **Invite**
6. Guest user account automatically added to directory

## Next Steps To Complete
1. Sign in to shanthiraj519@gmail.com inbox
2. Open email from **"Microsoft Invitations on behalf of [tenant]"**
3. Click **Accept invitation**
4. On **Permission requested by** page, click **Accept**
5. **My Apps** page opens → expect "There are no apps to show" (no apps assigned yet — this is normal)

## Verification
- Go back to **Entra ID** > **Users** in admin center
- Confirm guest user shows:
  - **User type**: Guest
  - **Invitation state**: Should change from *Pending Acceptance* → *Accepted* once you complete redemption

## Clean Up (after lab)
1. Entra ID > Users > select test guest user
2. Select **Delete user**

## Notes for AZ-500 exam relevance
- Minimum role needed: **Guest Inviter** OR **User Administrator**
- B2B invite emails from default onmicrosoft.com domain are subject to Exchange Online sending limits
- Next logical labs: assign guest to an app/group, test Conditional Access against guest, review guest access via **Access Reviews**

**Source:** [Microsoft Learn - Add a guest user and send an invitation](https://learn.microsoft.com/en-us/entra/external-id/b2b-quickstart-add-guest-users-portal)
