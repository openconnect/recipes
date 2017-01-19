# Duo authentication with ocserv

Two-factor authentication service from [Duo Security](https://duo.com/) can be
combined with ocserv. This recipe was tested on CentOS 7.

1.  Install ocserv >= 0.11.0

    Ensure PAM authentication is enabled in ocserv.conf. (This is the default.)

    ```
    auth = "pam"
    ```

2.  Install Duo Unix

    Duo provides [installation packages](https://duo.com/docs/duounix#linux-distribution-packages)

    Configure your site keys in /etc/duo/pam_duo.conf

3.  Configure PAM to enable Duo for password authentication

    Modify /etc/pam.d/password-auth:

    * Using local accounts (users in /etc/passwd)

        * Change auth pam_unix.so from "sufficient" to "requisite"
        * Add "auth sufficient pam_duo.so"

        ```
        --- password-auth.orig
        +++ password-auth
        @@ -1,6 +1,7 @@
         #%PAM-1.0
         auth        required      pam_env.so
        -auth        sufficient    pam_unix.so nullok try_first_pass
        +auth        requisite     pam_unix.so nullok try_first_pass
        +auth        sufficient    pam_duo.so
         auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
         auth        required      pam_deny.so
        ```

    * Using LDAP accounts and SSSD

        * Change auth pam_sss.so from "sufficient" to "requisite"
        * Add "auth sufficient pam_duo.so"

        ```
        --- password-auth-ac.orig
        +++ password-auth-ac
        @@ -3,7 +3,8 @@
         auth        [default=1 success=ok] pam_localuser.so
         auth        [success=done ignore=ignore default=die] pam_unix.so nullok try_first_pass
         auth        requisite     pam_succeed_if.so uid >= 1000 quiet_success
        -auth        sufficient    pam_sss.so forward_pass
        +auth        requisite     pam_sss.so forward_pass
        +auth        sufficient    pam_duo.so
         auth        required      pam_deny.so

         account     required      pam_unix.so broken_shadow
        ```

4.  You're done!

    ocserv passes the Duo prompts to VPN clients
