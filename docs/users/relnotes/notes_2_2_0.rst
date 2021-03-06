SSSD 2.2.0
==========


Highlights
----------

New features
^^^^^^^^^^^^
 * The Kerberos provider (and composite authentication providers based on it,
   like AD or IPA) can now include more KDC addresses or host
   names when writing data for the Kerberos locator plugin (see
   ``sssd_krb5_locator_plugin(8)``). This means that Kerberos client
   applications, such as ``kinit`` would be able to switch between multiple
   KDC servers discovered by SSSD. Please see description of the option
   ``krb5_kdcinfo_lookahead`` in the ``sssd-krb5(5)`` manual page for more
   information or refer to `the design page 
   <https://docs.pagure.org/SSSD.sssd/design_pages/kdcinfo_multiple_servers.html>`_
   (#3973, #3974, #3975)
 * The 2FA prompting can now be configured. The administrator can set custom
   prompts for first or second factor or select a single prompt for both
   factors. This can be configured per-service. Please see the section called
   "Prompting configuration" in the ``sssd.conf(5)`` manual page for more
   details or refer to `the design page 
   <https://docs.pagure.org/SSSD.sssd/design_pages/prompting_configuration.html>`_
   (#3264).
 * The LDAP authentication provider now allows to use a different method of
   changing LDAP passwords using a modify operation in addition to the default
   extended operation. This is meant to support old LDAP servers that do not
   implement the extended operation. The password change using the modification
   operation can be selected with ``ldap_pwmodify_mode = "ldap_modify"``. More
   information can also be found in `the design page 
   <https://docs.pagure.org/SSSD.sssd/design_pages/prompting_configuration.html>`_
   (#1314)
 * The ``auto_private_groups`` configuration option now takes a new value
   ``hybrid``. This mode autogenerates private groups for user entries where
   the UID and GID values have the same value and at the same time the GID
   value does not correspond to a real group entry in LDAP (#3822)
 * A new option ``ad_gpo_ignore_unreadable`` was added. This option,
   which defaults to false, can be used to ignore group policy containers in AD
   with unreadable or missing attributes. This is for the case when server
   contains GPOs that have very strict permissions on their attributes
   in AD but are unrelated to access control (#3867)
 * The ``cached_auth_timeout`` parameter is now inherited by trusted domains
   (#3960). The pre-authentication request is now cached as well when this
   option is in effect (#3960)
 * The ``ldap_sasl_mech`` option now accepts another mechanism ``GSS-SPNEGO``
   in addition to ``GSSAPI``. Using SPNEGO might be preferable with newer
   Active Directory servers especially with hardened configurations. SSSD might
   switch to using SPNEGO by default in a future release (#4006)
 * The ``sssctl`` tool has two new commands ``cert-show`` and ``cert-map``
   which can help in troubleshooting Smart-Card and in general user certificate
   related issues

Notable bug fixes
^^^^^^^^^^^^^^^^^
 * A potential race condition between SSSD receiving a notification to try
   switching to online mode and the network being actually reachable is
   now handled better. SSSD now tries to go online three times with an
   increasing delay between online checks up to 4s (#3467).
 * A potential deadlock in user resolution when the IPA provider fetches
   the keytab used to authenticate to a trusted AD domain was fixed (#3992)
 * When checking if objects that cannot be looked up exist locally and thus
   should be added to a negative cache with a longer negative TTL (see
   ``local_negative_timeout`` in ``sssd.conf(5)``), the blocking NSS API
   is no longer used. The blocking calls which might have caused a timeout
   especially during SSSD startup (#3963)
 * Some cache attributes used by the Kerberos ticket renewal code are
   now indexed, which speeds up the cache searches which might have otherwise
   caused SSSD to appear blocked and killed by the internal watchdog (#3968)
 * Cached objects from an Active Directory domain trusted by an IPA domain
   that no longer exist on the server are now properly removed from the
   cache (#3984)
 * The ``sudoRunAsUser/Group`` now work correctly with an IPA configuration
   that also uses the ``domain_resolution_order``, either set locally or
   centrally (#3957)
 * Certificates that are completely missing the Key Usage (KU) certificate
   extension are now handled gracefully (rhbz#1660899)
 * The sudo smart refresh (see man ``sssd-sudo``) now correctly uses the 
   highest USN number, which results in more efficient queries (#3997)
 * The ``pam_sss`` module now returns PAM_USER_UNKNOWN if the PAM socket
   is missing completely. This could have been the case if SSSD is running
   with the files domain only and a user resolved by a completely different
   PAM module logs in (#3988)
 * Netgroups lookups now honor the midpoint refresh interval set by
   ``cache_refresh_percent`` (#3947)
 * The list of users or groups from the ``filter_users/filter_groups`` lists
   which will be negatively cached, avoiding lookups of those entries, are
   now correctly evaluated for domains that are discovered after sssd
   had started (#3983). These lists can also now include UPNs (#3978)
 * The IPA access provider no longer fails if the configuration file
   completely disables dereference by setting ``ldap_deref_threshold=0``
   (#3979)
 * The ``sss_cache`` tool does not print loud warnings in case the sssd
   cache cannot be written to, typically this was occuring when ``/var``
   was mounted read-only during an ``rpm-ostree`` update.
 * The command line tools such as ``sssctl`` can now operate on the implicit
   files domain (#3769)
 * The files and proxy provider no longer crash on receiving a request
   to go online, which they don't implement (#4014)
 * A potential crash in the online check callback was fixed (#3990)
 * The winbind ID-mapping plugin now works with recent Samba releases again
   (#4005)

Packaging Changes
-----------------
None

Documentation Changes
---------------------
 * A new option ``ad_gpo_ignore_unreadable`` was added
 * A new option ``krb5_kdcinfo_lookahead`` was added
 * A new option ``ldap_pwmodify_mode`` was added
 * The option ``ldap_sasl_mech`` now accepts a new value ``GSS-SPNEGO``
 * The option ``auto_private_groups`` now accepts a new value ``hybrid``
 * Multi-factor prompting can now be configured in a separate section called
   ``[prompting]``

Tickets Fixed
-------------
 * `4016 <https://pagure.io/SSSD/sssd/issue/4016>`_ - sssd fails to build with Python 3.8
 * `4015 <https://pagure.io/SSSD/sssd/issue/4015>`_ - The server error message is not returned if password change fails
 * `4014 <https://pagure.io/SSSD/sssd/issue/4014>`_ - The files provider does not handle resetOffline properly
 * `4006 <https://pagure.io/SSSD/sssd/issue/4006>`_ - sssd does not properly check GSS-SPNEGO
 * `3997 <https://pagure.io/SSSD/sssd/issue/3997>`_ - sudo: always use server highest usn for smart refresh
 * `3992 <https://pagure.io/SSSD/sssd/issue/3992>`_ - ipa-getkeytab can call NSS operation which might deadlock the subdomains request
 * `3991 <https://pagure.io/SSSD/sssd/issue/3991>`_ - providers/data_provider_be: code review required
 * `3990 <https://pagure.io/SSSD/sssd/issue/3990>`_ - providers/data_provider_be: potential dereferencing of 'bad' ptr
 * `3989 <https://pagure.io/SSSD/sssd/issue/3989>`_ - Consider merge of two "negcache" tests.
 * `3988 <https://pagure.io/SSSD/sssd/issue/3988>`_ - pam_sss failing for external users not configured via sssd
 * `3984 <https://pagure.io/SSSD/sssd/issue/3984>`_ - IPA: Deleted user from trusted domain is not removed properly from the cache on IPA clients
 * `3983 <https://pagure.io/SSSD/sssd/issue/3983>`_ - filter_users option is not applied to sub-domains if SSSD starts offline
 * `3980 <https://pagure.io/SSSD/sssd/issue/3980>`_ - sudorule matching when no host or hostcat set
 * `3979 <https://pagure.io/SSSD/sssd/issue/3979>`_ - The HBAC code requires dereference to be enabled and fails otherwise
 * `3978 <https://pagure.io/SSSD/sssd/issue/3978>`_ - UPN negative cache does not use values from 'filter_users' config option
 * `3976 <https://pagure.io/SSSD/sssd/issue/3976>`_ - crash in dp_failover_active_server
 * `3975 <https://pagure.io/SSSD/sssd/issue/3975>`_ - Lookahead resolving of host names to provide names for the kdcinfo plugin
 * `3974 <https://pagure.io/SSSD/sssd/issue/3974>`_ - Write a list of host names up to a configurable limit to the kdcinfo files
 * `3973 <https://pagure.io/SSSD/sssd/issue/3973>`_ - The kdcinfo plugin should be able to resolve host names
 * `3972 <https://pagure.io/SSSD/sssd/issue/3972>`_ - Circular dependency between subdomains update and NSS responder invoking getDomains
 * `3968 <https://pagure.io/SSSD/sssd/issue/3968>`_ - krb5_child_init: check_ccache_files() might be *too* slow with large cache
 * `3965 <https://pagure.io/SSSD/sssd/issue/3965>`_ - [RFE]: Optionally disable generating auto private groups for subdomains of an AD provider
 * `3964 <https://pagure.io/SSSD/sssd/issue/3964>`_ - Responders: `is_user_local_by_name()` should avoid calling nss API entirely
 * `3963 <https://pagure.io/SSSD/sssd/issue/3963>`_ - Responders: processing of `filter_users`/`filter_groups` should avoid calling blocking NSS API
 * `3960 <https://pagure.io/SSSD/sssd/issue/3960>`_ - cached_auth_timeout not honored for AD users authenticated via trust with FreeIPA
 * `3957 <https://pagure.io/SSSD/sssd/issue/3957>`_ - sudo: runAsUser/Group does not work with domain_resolution_order
 * `3946 <https://pagure.io/SSSD/sssd/issue/3946>`_ - SSSD netgroups do not honor entry_cache_nowait_percentage
 * `3931 <https://pagure.io/SSSD/sssd/issue/3931>`_ - proxy provider is not working with enumerate=true when trying to fetch all groups
 * `3907 <https://pagure.io/SSSD/sssd/issue/3907>`_ - responders chain requests that were issued before reconnection to sssd_be
 * `3899 <https://pagure.io/SSSD/sssd/issue/3899>`_ - change the default service search base in SSSD-IPA
 * `3867 <https://pagure.io/SSSD/sssd/issue/3867>`_ - [RFE] Need an option in SSSD so that it will skip GPOs that have groupPolicyContainers, unreadable by SSSD.
 * `3861 <https://pagure.io/SSSD/sssd/issue/3861>`_ - Python multihost tests are not part of upstream tarball
 * `3838 <https://pagure.io/SSSD/sssd/issue/3838>`_ - KCM: If the default ccache cannot be found, fall back to the first one
 * `3822 <https://pagure.io/SSSD/sssd/issue/3822>`_ - Enable generating user private groups only for users with no primary GID
 * `3769 <https://pagure.io/SSSD/sssd/issue/3769>`_ - sssd tools don't handle the implicit domain
 * `3636 <https://pagure.io/SSSD/sssd/issue/3636>`_ - nested group missing after updates on provider
 * `3614 <https://pagure.io/SSSD/sssd/issue/3614>`_ - FIPS mode breaks using pysss.so (sss_obfuscate)
 * `3467 <https://pagure.io/SSSD/sssd/issue/3467>`_ - online detection in case sssd starts before network does appears to be broken
 * `3401 <https://pagure.io/SSSD/sssd/issue/3401>`_ - sssd does not failover to another IPA server if just the KDC service fails
 * `3264 <https://pagure.io/SSSD/sssd/issue/3264>`_ - [RFE] Make 2FA prompting configurable
 * `1314 <https://pagure.io/SSSD/sssd/issue/1314>`_ - RFE Request for allowing password changes using SSSD in DS which dont follow OID's from RFC 3062

Detailed changelog
------------------

* Alexey Tikhonov (24): 

      * negcache: avoid "is_*_local" calls in some cases 
      * providers/ldap: sdap_extend_map_with_list() fixed 
      * providers/ldap: const params should be const 
      * providers/proxy: small optimization 
      * providers/proxy: fixed wrong check 
      * providers/proxy: fixed usage of wrong mem ctx 
      * providers/proxy: got rid of excessive mem copies 
      * providers/proxy: fixed erroneous free of orig_grp 
      * providers/proxy: const params should be const 
      * Util: added facility to load nss lib syms 
      * responder/negcache: avoid calling nsswitch NSS API 
      * negcache_files: got rid of large array on stack 
      * TESTS: moved cwrap/test_negcache to cmocka tests 
      * TESTS: fixed regression in cmocka/test_negcache_2.c 
      * ci/sssd.supp: getpwuid() leak suppression 
      * data_provider_be: fixed dereferencing of 'bad' ptr 
      * TESTS: two `negcache` tests were merged 
      * data_provider_be: got rid of went_offline usage 
      * providers/ipa: Fixed obvious copy-paste error 
      * providers/ipa: Changed default service search base 
      * TESTS: ability to run unit tests under valgrind 
      * Monitor & utils: got rid of pid filename duplication 
      * Monitor: fixed bug with services launch 
      * ldap/sdap_idmap.c: removed unnecessary include 

* Branen Salmon (1): 

      * knownhostsproxy: friendly error msg for NXDOMAIN 

* Colin Walters (1): 

      * sss_cache: Do nothing if SYSTEMD_OFFLINE=1 

* Jakub Hrozek (21): 

      * Updating the version to track the next release 
      * TESTS: Add a unit test for UPNs stored by sss_ncache_prepopulate 
      * UTIL: Add a is_domain_mpg shorthand 
      * UTIL: Convert bool mpg to an enum mpg_mode 
      * CONFDB: Read auto_private_groups as string, not bool 
      * CONFDB/SYSDB: Add the hybrid MPG mode 
      * CACHE_REQ: Add cache_req_data_get_type() 
      * NSS: Add the hybrid-MPG mode 
      * TESTS: Add integration tests for auto_private_groups=hybrid 
      * SYSDB: Inherit cached_auth_timeout from the main domain 
      * AD: Allow configuring auto_private_groups per subdomain or with subdomain_inherit 
      * SDAP: Add sdap_has_deref_support_ex() 
      * IPA: Use dereference for host groups even if the configuration disables dereference 
      * KCM: Fall back to using the first ccache if the default does not exist 
      * krb5: Do not use unindexed objectCategory in a search filter 
      * SYSDB: Index the ccacheFile attribute 
      * krb5: Silence an error message if no cache entries have ccache stored but renewal is enabled 
      * PAM: Also cache SSS_PAM_PREAUTH 
      * LDAP: Return the error message from the extended operation password change also on failure 
      * Update the translations for the 2.2.0 release 
      * Updating the version for the 2.2.0 release 

* Michal Židek (2): 

      * GPO: Add option ad_gpo_ignore_unreadable 
      * tests: Add multihost tests to upstream tarball 

* Mikhail Novosyolov (1): 

      * Fix pidpath in systemd unit 

* Niranjan M.R (7): 

      * TESTS: Add @Title to test case docstrings for basic sanity tests 
      * TESTS: Add @Title to test case docstrings for config tests 
      * TESTS: Add @Title to test case docstrings for KCM tests. 
      * TESTS: Add @Title to test case docstrings for sssctl config tests. 
      * TESTS: Add @Title to test case docstrings for sudo tests 
      * TESTS: Add @Title to test case docstrings for files tests. 
      * TESTS: Add @Title to test case docstrings for ifp tests 

* Pavel Březina (18): 

      * netgroups: honor cache_refresh_percent 
      * sdap: add sdap_modify_passwd_send 
      * sdap: add ldap_pwmodify_mode option 
      * sdap: split password change to separate request 
      * sdap: use ldap_pwmodify_mode to change password 
      * be: remember last good server's name instead of fo_server structure 
      * sudo ipa: do not store rules without sudoHost attribute 
      * ipa: store sudo runas attribute with internal fqname 
      * sudo: format runas attributes to correct output name 
      * memberof: keep memberOf attribute for nested member 
      * sudo: always use server highest known usn for smart refresh 
      * man: update sudo smart refresh documentation to reflect new USN behavior 
      * ci: do not fail everything when one distro fails 
      * ci: archive test-suite.log 
      * ci: add Fedora 30 
      * ci: remove code duplication in Jenkinsfile 
      * ci: run moderate set of tests 
      * ci: do not install dependencies 

* Samuel Cabrero (1): 

      * SUDO: Allow defaults sudoRole without sudoUser attribute 

* Sumit Bose (29): 

      * NEGCACHE: initialize UPN negative cache as well 
      * NEGCACHE: fix typo in debug message 
      * NEGCACHE: repopulate negative cache after get_domains 
      * ldap: add users_get_handle_no_user() 
      * ldap: make groups_get_handle_no_group() public 
      * ipa s2n: fix typo 
      * ipa s2n: do not add UPG member 
      * ipa s2n: try to remove objects not found on the server 
      * pam_sss: PAM_USER_UNKNOWN if socket is missing 
      * pam: introduce prompt_config struct 
      * authtok: add dedicated type for 2fa with single string 
      * pam_sss: use configured prompting 
      * PAM: add initial prompting configuration 
      * intg: add test for password prompt configuration 
      * ipa: ipa_getkeytab don't call libnss_sss 
      * winbind idmap plugin: update struct idmap_domain to latest version 
      * sdap: update last_usn on reconnect 
      * SDAP: allow GSS-SPNEGO for LDAP SASL bind as well 
      * sdap: inherit SDAP_SASL_MECH if not set explicitly 
      * DP: add NULL check to be_ptask_{enable|disable} 
      * certmap: allow missing KU in OpenSSL version 
      * test: add certificate without KU to certmap tests 
      * certmap: add sss_certmap_display_cert_content() 
      * sssctl: add cert-show 
      * files: add missing newline to debug message 
      * sssctl: add cert-map 
      * tests: fix enctypes in test_copy_keytab 
      * CI: use python3-pep8 on Fedora 31 and later 
      * BUILD: fix libpython handling in Python3.8 

* Tom Briden (1): 

      * build: only do automagic linking against systemd if required 

* Tomas Halman (6): 

      * krb5_locator: Allow hostname in kdcinfo files 
      * krb5: Write multiple dnsnames into kdc info file 
      * Providers: Delay online check on startup 
      * krb5: Lookahead resolving of host names 
      * sss_cache: Do nothing if /var is read-only 
      * confdb: sssd tools don't handle the implicit domain 

* Tomislav Dukaric (1): 

      * self.OPTCRE.match(line) fails if there's a whitespace before option name, which is valid for SSSD. This will ignore any whitespace before the option 

* Yuri Chornoivan (1): 

      * Fix various minor typos 

* realsobek (1): 

      * fix man page reference 
