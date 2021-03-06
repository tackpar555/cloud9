.. Copyright 2010-2018 Amazon.com, Inc. or its affiliates. All Rights Reserved.

   This work is licensed under a Creative Commons Attribution-NonCommercial-ShareAlike 4.0
   International License (the "License"). You may not use this file except in compliance with the
   License. A copy of the License is located at http://creativecommons.org/licenses/by-nc-sa/4.0/.

   This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND,
   either express or implied. See the License for the specific language governing permissions and
   limitations under the License.

.. _credentials:

#####################################################
Calling AWS Services from an Environment in |AC9long|
#####################################################

.. meta::
    :description:
        Provides guidance for configuring an environment in AWS Cloud9 to interact with AWS services.

You can call AWS services from an |envfirst|. For example, you can:

* Upload and download data in |S3long| (|S3|) buckets.
* Send broadcast notifications through |SNSlong| (|SNS|) topics.
* Read and write data in |DDBlong| (|DDB|) databases.

You can call AWS services from your |env| in several ways. For example, you can use the |clilong|
(|cli|) to run commands from
a terminal session. You can also call AWS services from code you run within your |env|, using AWS SDKs for programming languages such as JavaScript, Python, Ruby, PHP, Go, and C++.
For more information, see the |cli-ug|_ and `AWS SDKs <https://aws.amazon.com/tools/#sdk>`_.

Each time the |cli| or your code calls an AWS service, the |cli| or your code must provide a set of AWS access credentials along with the call. These credentials determine whether the caller has the appropriate permissions to make the call. If the
credentials don't cover the appropriate permissions, the call will fail. 

There are several ways to provide credentials to your |env|. The following table describes some approaches.

.. list-table::
   :widths: 1 3
   :header-rows: 1

   * - |envtitle| type
     - Approach
   * - |EC2|
     - Use |AC9tempcreds|.

       We recommend this approach for an |envec2|. |AC9tempcreds| manage AWS access credentials in an |envec2| on your behalf, while also following AWS security best practices.

       **If you are using an EC2 environment, you can skip the rest of this topic, as AWS managed temporary credentials are already set up for you in the environment.**

       For more information, see :ref:`auth-and-access-control-temporary-managed-credentials`.
   * - |EC2|
     - Attach an |IAM| instance profile to the instance.

       You should only use this approach if for some reason you can't use |AC9tempcreds|. Similar to |AC9tempcreds|,
       an instance profile manages AWS access credentials on your behalf. However, you must create, manage, and attach
       the instance profile to the |EC2| instance yourself.

       For instructions, see :ref:`credentials-temporary`.
   * - |EC2| or SSH
     - Store your permanent AWS access credentials within the |env|.

       This approach is less secure than using temporary AWS access credentials. However, it is the only supported approach for an |envssh|.

       For instructions, see :ref:`credentials-permanent-create`.
   * - |EC2| or SSH
     - Insert your permanent AWS access credentials directly into your code.

       We discourage this approach because it doesn't follow AWS security best practices.

       Because we discourage this approach, we do not cover it in this topic.

.. _credentials-temporary:

Create and Use an Instance Profile to Manage Temporary Credentials
==================================================================

.. note:: You cannot use this procedure for an |envfirstssh|. Instead, skip ahead to :ref:`credentials-permanent-create`.

   We recommend using |AC9tempcreds| instead of an instance profile. Follow these instructions only if for some reason you cannot use |AC9tempcreds|.
   For more information, see :ref:`auth-and-access-control-temporary-managed-credentials`.

In this procedure, you will use the |IAM| and |EC2| consoles to create and attach an |IAM| instance profile to the |EC2| instance that connects to your |env|. This instance profile will manage
temporary credentials on your behalf. This procedure assumes you have already created a |env| in |AC9|. To create a |env|, see :doc:`Create an Environment <create-environment>`.

.. topic:: To create an instance profile

   .. note:: If you already have an |IAM| role that contains an instance profile, skip ahead to the next
      procedure, **To attach an instance profile to an instance**.

   #. Sign in to the |IAM| console, at https://console.aws.amazon.com/iam.

      For this step, we recommend you sign in using credentials for an |IAM| administrator user in your AWS account. If you cannot 
      do this, check with your AWS account administrator.

   #. In the navigation bar, choose :guilabel:`Roles`.

      .. note:: You cannot use the |IAM| console to create an instance profile by itself. You must create an |IAM| role, which contains an instance profile.

   #. Choose :guilabel:`Create role`.
   #. On the :guilabel:`Select type of trusted entity` page, with :guilabel:`AWS service` already chosen, for :guilabel:`Choose the service that will use this role`, choose :guilabel:`EC2`.
   #. For :guilabel:`Select your use case`, choose :guilabel:`EC2`.
   #. Choose :guilabel:`Next: Permissions`.
   #. On the :guilabel:`Attach permissions policies` page, in the list of policies, select the box next to :guilabel:`AdministratorAccess`, and then choose :guilabel:`Next: Review`.

      .. note:: The :guilabel:`AdministratorAccess` policy allows unrestricted access to all AWS actions and resources across your AWS account. It should be used only for experimentation purposes.
         For more information, see :IAM-ug:`IAM Policies <access_policies>` in the |IAM-ug|.

   #. On the :guilabel:`Review` page, for :guilabel:`Role Name`, type a name for the role (for example :kbd:`my-demo-cloud9-instance-profile`).
   #. Choose :guilabel:`Create Role`.

   To attach the role's instance profile to the instance, see the next procedure, **To attach an instance
   profile to an instance**.

.. topic:: To attach an instance profile to an instance

   #. Sign in to the |EC2| console, at https://console.aws.amazon.com/ec2.

      For this step, we recommend you sign in using credentials for an |IAM| administrator user in your AWS account. If you cannot 
      do this, check with your AWS account administrator.

   #. In the navigation bar, be sure the region selector displays the AWS Region that matches the one
      for your |env|. For example, if you created your |env| in the
      US East (Ohio) region, choose :guilabel:`US East (Ohio)` in the region selector here as well.
   #. Choose the :guilabel:`Running Instances` link or, in the navigation pane, expand :guilabel:`Instances`, and then choose :guilabel:`Instances`.
   #. In the list of instances, choose the instance with the :guilabel:`Name` that includes your |env| name. For example, if your |env| name is :code:`my-demo-environment`, choose the
      instance with the :guilabel:`Name` that includes :guilabel:`my-demo-environment`.
   #. Choose :menuselection:`Actions, Instance Settings, Attach/Replace IAM Role`.

      .. note:: Although you are attaching a role to the instance, the role contains an instance profile.

   #. On the :guilabel:`Attach/Replace IAM Role` page, for :guilabel:`IAM role`, choose the name of the role you identified or that you created in the previous procedure, and then choose :guilabel:`Apply`.
   #. Back in the |env|, use the |cli| to run the :code:`aws configure` command. Do not specify any values for :guilabel:`AWS Access Key ID` or
      :guilabel:`AWS Secret Access Key` (press :kbd:`Enter` after each of these prompts). For :guilabel:`Default region name`, specify the AWS Region closest to you or the region where your AWS resources are located.
      For example, :code:`us-east-2` for the US East (Ohio) Region. For a list of regions, see :AWS-gr:`AWS Regions and Endpoints <rande>` in the |AWS-gr|.
      Optionally, specify a value for :guilabel:`Default output format` (for example, :code:`json`).

You can now start calling AWS services from your |env|. To use the |cli| to call AWS services, see the :doc:`AWS CLI Sample <sample-aws-cli>`. To call AWS services from your code, see our other :doc:`samples <samples>`.

.. _credentials-permanent-create:

Create and Store Permanent Access Credentials in an |envtitle|
==============================================================

.. note:: If you are using an |envfirstec2|, we recommend you use |AC9tempcreds| instead of AWS permanent access credentials. To work with |AC9tempcreds|,
  see :ref:`auth-and-access-control-temporary-managed-credentials`.

.. topic:: To create permanent access credentials

   In this procedure, you use the |IAMlong| (|IAM|) console to generate a set of permanent credentials that the |cli| or your code can use when calling AWS services.
   This set includes an AWS access key ID and an AWS secret access key, which are unique to your user in your AWS account. If you already have
   an AWS access key ID and an AWS secret access key, note those credentials, and then skip ahead to the next procedure,
   **To store permanent access credentials in an environment**.

   #. Sign in to the |IAM| console, at https://console.aws.amazon.com/iam.

      For this step, we recommend you sign in using credentials for an |IAM| administrator user in your AWS account. If you cannot 
      do this, check with your AWS account administrator.
      
   #. In the navigation bar, choose :guilabel:`Users`.
   #. In the list of users, choose the name of the user you created or identified in :doc:`Team Setup <setup>`.
   #. Choose the :guilabel:`Security credentials` tab.
   #. For :guilabel:`Access keys`, choose :guilabel:`Create access key`.
   #. In the :guilabel:`Create access key` page, choose :guilabel:`Show`, and make a note of the :guilabel:`Access key ID` and :guilabel:`Secret access key` values.
      We recommend you also choose :guilabel:`Download .csv file` and save these credentials in a secure location.

   To store your permanent access credentials in an |env|, see the next procedure, **To store permanent
   access credentials in an environment**.

.. topic:: To store permanent access credentials in an |env|

   In this procedure, you use the |AC9IDE| to store your permanent AWS access credentials in your |env|. This procedure assumes you have already created an |env| in |AC9|,
   opened the |env|, and are displaying the |AC9IDE| in your web browser. For more information, see :doc:`Creating an Environment <create-environment>` and :doc:`Opening an Environment <open-environment>`.

   .. note:: The following procedure shows how to store your permanent access credentials by using environment variables.
      If you have the |cli| installed in your |env|, you can
      use the :command:`aws configure` command to store your permanent access credentials instead. For instructions, see 
      :cli-user-guide:`Quick Configuration <cli-chap-getting-started.html#cli-quick-configuration>` in the |cli-ug|.

   #. With your |env| open, in the |AC9IDE|, start a new terminal session, if one is not already started. To start a new terminal session, on the
      menu bar, choose :menuselection:`Window, New Terminal`.
   #. Run each of the following commands, one command at a time, to set local environment variables representing your permanent access credentials.
      In these commands, :samp:`{YOUR-ACCESS-KEY-ID}` is your AWS access key ID, :samp:`{YOUR-SECRET-ACCESS-KEY}` is your
      AWS secret access key, and :samp:`{YOUR-DEFAULT-REGION-ID}` is the AWS Region identifier associated with the AWS Region closest to you (or your preferred AWS Region).
      For a list of available identifiers, see :AWS-gr:`AWS Regions and Endpoints <rande>` in the |AWS-gr|. For example, for the US East (Ohio) Region, you would use
      :kbd:`us-east-2`.

      .. code-block:: sh

         export AWS_ACCESS_KEY_ID=YOUR-ACCESS-KEY-ID
         export AWS_SECRET_ACCESS_KEY=YOUR-SECRET-ACCESS-KEY
         export AWS_DEFAULT_REGION=YOUR-DEFAULT-REGION-ID

   #. Note that the preceding environment variables are valid only for the current terminal session. To make these environment variables available across terminal sessions, you must add them
      to your shell profile file as user environment variables. To do this, do the following:

      #. In the :guilabel:`Environment` window of the |IDE|, choose the gear icon, and then choose :guilabel:`Show Home in Favorites`.
         Repeat this step and choose :guilabel:`Show Hidden Files` as well.
      #. Open the :file:`~/.bashrc` file.
      #. Type or paste the following code at the end of the file. In these commands, :samp:`{YOUR-ACCESS-KEY-ID}` is your AWS access key ID, :samp:`{YOUR-SECRET-ACCESS-KEY}` is your
         AWS secret access key, and :samp:`{YOUR-DEFAULT-REGION-ID}` is the AWS Region identifier associated
         with the AWS Region closest to you (or your preferred AWS Region).
         For a list of available identifiers, see :AWS-gr:`AWS Regions and Endpoints <rande>` in the |AWS-gr|. (For example, for the US East (Ohio) Region, you would use
         :kbd:`us-east-2`.)

         .. code-block:: sh

            export AWS_ACCESS_KEY_ID=YOUR-ACCESS-KEY-ID
            export AWS_SECRET_ACCESS_KEY=YOUR-SECRET-ACCESS-KEY
            export AWS_DEFAULT_REGION=YOUR-DEFAULT-REGION-ID

      #. Save the file.
      #. Source the :file:`~/.bashrc` file to load these new environment variables.

         .. code-block:: sh

            . ~/.bashrc

   You can now start calling AWS services from your |env|. To use the |cli| to call AWS services, see the :doc:`AWS CLI Sample <sample-aws-cli>`. To call AWS services from your code, see our other :doc:`samples <samples>`.
