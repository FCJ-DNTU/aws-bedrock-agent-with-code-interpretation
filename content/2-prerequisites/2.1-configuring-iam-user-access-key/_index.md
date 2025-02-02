---
title: "Configuring IAM User Access Key"
weight: 1
chapter: false
pre: " <b> 2.1 </b> "
---

#### Step 1: Log in to the AWS Management Console
Log in to the AWS Management Console using your Root User or an account with administrative privileges.

#### Step 2: Access the Identity and Access Management (IAM) Service
After successfully logging in, navigate to the **Identity and Access Management** (IAM) service from the main Console interface.

![iam-interface](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image.png)

#### Step 3: Create a New User
1. In the IAM interface, go to the **Users** tab and select **Create user**.
2. In the **Specify user details** section, enter the username in the **User name** field.
3. Select **Provide user access to the AWS Management Console**.
4. Under **Console Password**, choose **Custom Password** and enter the desired password for the user.
5. Uncheck **User must create a new password at next sign-in** if you do not require the user to change their password upon first login.
6. Select **Next** to continue.

![step-1](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image-1.png)

#### Step 4: Configure Access Permissions
1. In the **Set Permissions** interface, select the `AdministratorAccess` policy to grant full access to the user.
2. Select **Next** to continue.

![step-2](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image-2.png)

#### Step 5: Confirm User Creation
Review the results and confirm that the user has been successfully created.

![step-4](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image-3.png)

#### Step 6: Create an Access Key for the User
1. Click on the newly created user.
2. Navigate to the **Security Credentials** tab.
3. Under **Access Keys**, select **Create access key**.

![manage-access-key](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image-4.png)

#### Step 7: Save the Access Key and Secret Key
1. In the **Access key best practices & alternatives** interface, select the use case **Command Line Interface (CLI)**.
2. Select **Next** to continue.

![use-cases](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image-5.png)

3. Save the **Access Key** and **Secret Key** displayed in the **Retrieve Access Key** interface for use in related applications or tools.

![sucessfully-created](/images/2-prerequisites/2.1-configuring-iam-user-access-key/image-6.png)