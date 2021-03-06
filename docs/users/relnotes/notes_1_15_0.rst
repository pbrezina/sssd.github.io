SSSD 1.15.0
===========

Highlights
----------
 * SSSD now allows the responders to be activated by the systemd service
   manager and exit when idle. This means the ``services`` line in sssd.conf
   is optional and the responders can be started on-demand, simplifying the
   sssd configuration. Please note that this change is backwards-compatible
   and the responders listed explicitly in sssd.conf's services line are
   managed by sssd in the same manner as in previous releases. Please refer to
   ``man sssd.conf(5)`` for more information
 * The sudo provider is no longer disabled for configurations that do not
   explicitly include the ``sudo`` responder in the ``services`` list. In order
   to disable the sudo-related back end code that executes the periodic LDAP
   queries, set the ``sudo_provider`` to ``none`` explicitly
 * The watchdog signal handler no longer uses signal-unsafe functions. This
   bug was causing a deadlock in case the watchdog was about to kill a
   stuck process
 * A bug that prevented TLS to be set up correctly on systems where libldap
   links with GnuTLS was fixed
 * The functionality to alter SSSD configuration through the D-Bus interface
   provided by the IFP responder was removed. This functionality was
   not used to the best of our knowledge, had no tests and prevented the
   InfoPipe responder from running as a non-privileged user.
 * A bug that prevented statically-linked applications from using libnss_sss
   was fixed by removing dependency on ``-lpthreads`` from the ``libnss_sss``
   library (please see `<https://sourceware.org/bugzilla/show_bug.cgi?id=20500>`_
   for an example on why linking with ``-lpthread`` from an NSS modules
   is problematic)
 * Previously, SSSD did not ignore GPOs that were missing the
   gPCFunctionalityVersion attribute and failed the whole GPO
   processing. Starting with this version, the GPOs without the
   gPCFunctionalityVersion are skipped.

Packaging Changes
-----------------
 * The Augeas development libraries are no longer required since the
   configuration manipulation interface was dropped from the InfoPipe responder
 * The libsss_config.so internal library was removed as well due to removal
   of the InfoPipe config management
 * In order to manage socket-activated or bus activated responders,
   each responder is now represented by a systemd service file
   (e.g. sssd-nss.service). All responders except InfoPipe, which is
   bus-activated, are also managed by a socket unit file (e.g. sssd-nss.socket)

Documentation Changes
---------------------
 * The sssd-secrets responder gained a new option max_payload_size that
   allows the administrator to limit the maximum size of a secret
 * A new option responder_idle_timeout was added to support idle termination of socket-activated responders
 * The sssd-ad and sssd-ipa man pages now summarize differences between
   the generic Kerberos/LDAP back end and the specialized IPA/AD back ends

Tickets Fixed
-------------
 * `#697 <https://pagure.io/SSSD/sssd/issue/697>`_ - Use command line arguments instead env vars for krb5_child 
 * `#201 <https://pagure.io/SSSD/sssd/issue/201>`_ - Man pages do not specify that sssd dyndns_refresh_interval < 60 is pulled to 60 seconds 
 * `#243 <https://pagure.io/SSSD/sssd/issue/243>`_ - [RFE] Socket-activate responders 
 * `#517 <https://pagure.io/SSSD/sssd/issue/517>`_ - krb5_child: Remove getenv() ran as root 
 * `#060 <https://pagure.io/SSSD/sssd/issue/060>`_ - better debugging of timestamp cache modifications 
 * `#129 <https://pagure.io/SSSD/sssd/issue/129>`_ - [RFE] socket-activate the IFP responder 
 * `#151 <https://pagure.io/SSSD/sssd/issue/151>`_ - cache_req: complete the needs of NSS responders 
 * `#156 <https://pagure.io/SSSD/sssd/issue/156>`_ - nss_sss might leak memory when calling thread goes away 
 * `#214 <https://pagure.io/SSSD/sssd/issue/214>`_ - Update man pages for any AD provider config options that differ from ldap/krb5 providers defaults 
 * `#215 <https://pagure.io/SSSD/sssd/issue/215>`_ - Review and update SSSD's wiki pages for 1.15 Alpha release 
 * `#235 <https://pagure.io/SSSD/sssd/issue/235>`_ - SSSCTL should not be case sensitive when searching for usernames or groups in a case-insensitive domain 
 * `#245 <https://pagure.io/SSSD/sssd/issue/245>`_ - [RFE] Shutdown timeout for {socket,bus}-activated responders 
 * `#275 <https://pagure.io/SSSD/sssd/issue/275>`_ - Unchecked return value of sss_cmd_empty_packet(pctx->creq->out); 
 * `#283 <https://pagure.io/SSSD/sssd/issue/283>`_ - getsidbyid can fail in some cases due to cache_req refactoring 
 * `#284 <https://pagure.io/SSSD/sssd/issue/284>`_ - getsidbyname does not work properly with case insensitive domains

Detailed Changelog
------------------
 * Amith Kumar (1):

   * MAN: Updation of sssd-ad man page for case when dyndns_refresh_interval < 60 seconds

 * Carl Henrik Lunde (1):

   * Prevent use after free in fd_input_available

 * David Michael (1):

   * BUILD: Find a host-prefixed krb5-config when cross-compiling

 * Fabiano Fidêncio (34):

   * SECRETS: Fix secrets rule in the allowed sections
   * SECRETS: Add allowed_sec_users_options
   * SECRETS: Delete all secrets stored during "max_secrets" test
   * SECRETS: Add configurable payload size limit of a secret
   * BUILD: Drop libsss_config
   * IFP: Remove "ChangeDebugTemporarily" method
   * AUTOFS: Check return of sss_cmd_empty_packet()
   * SUDO: Drop logic to disable the backend in case the provider is not set
   * MONITOR: Expose the monitor's services type
   * MONITOR: Pass the service type to the RegisterService method
   * UTIL: Introduce --socket-activated cmdline option for responders
   * UTIL: Introduce --dbus-activated cmd option for responders
   * RESPONDER: Make responders' common code ready for socket activation
   * AUTOFS: Make AutoFS responder socket-activatable
   * NSS: Make NSS responder socket-activatable
   * PAC: Make PAC responder socket-activatable
   * PAM: Make PAM responder socket-activatable
   * SSH: Make SSH responder socket-activatable
   * SUDO: Make Sudo responder socket-activatable
   * IFP: Make IFP responder dbus-activatable
   * MONITOR: Split up check_services()
   * MONITOR: Deal with no services set up
   * MONITOR: Deal with socket-activated responders
   * MAN: Mention that the services' list is optional
   * MAN: "user" doesn't work with socket-activated services
   * MONITOR: Don't expose monitor_common_send_id()
   * SBUS: Add a time_t pointer to the sbus_connection
   * SBUS: Add destructor data to sbus_connection
   * RESPONDER: Make clear ``{reset_,}idle_timer()`` are related to client
   * RESPONDER: Don't expose client_idle_handler()
   * RESPONDER: Shutdown {dbus,socket}-activated responders in case they're idle
   * RESPONDER: Change how client timeout is calculated
   * SERVER: Set the process group during server_setup()
   * WATCHDOG: Avoid non async-signal-safe from the signal_handler

 * Howard Guo (1):

   * sss_client: Defer thread cancellation until completion of nss/pam operations

 * Jakub Hrozek (16):

   * Updating the version for the 1.14.3 development
   * Updating the version to track sssd-1-15 development
   * SYSDB: Split sysdb_try_to_find_expected_dn() into smaller functions
   * SYSDB: Augment sysdb_try_to_find_expected_dn to match search base as well
   * MONITOR: Do not set up watchdog for monitor
   * MONITOR: Remove deprecated pong sbus method
   * MONITOR: Remove unused shutDown sbus method
   * Qualify ghost user attribute in case ldap_group_nesting_level is set to 0
   * tests: Add a test for group resolution with ldap_group_nesting_level=0
   * BUILD: Fix a typo in inotify.m4
   * SSH: Use default_domain_suffix for users' authorized keys
   * SYSDB: Suppress sysdb_delete_ts_entry failed: 0
   * STAP: Only print transaction statistics if the script caught some transactions
   * test_sssctl: Add an integration test for sssctl netgroup-show
   * KRB5: Advise the user to inspect the krb5_child.log if the child fails with a System Error
   * IFP: Fix GetUserAttr

 * Justin Stephenson (2):

   * MAN: Document different defaults for AD provider
   * MAN: Document different defaults for IPA provider

 * Lukas Slebodnik (45):

   * crypto: Port libcrypto code to openssl-1.1
   * BUILD: Fix build without samba
   * libcrypto: Check right value of CRYPTO_memcmp
   * crypto-tests: Add unit test for sss_encrypt + sss_decrypt
   * crypto-tests: Rename encrypt decrypt test case
   * BUILD: Accept krb5 1.15 for building the PAC plugin
   * dlopen-test: Use portable macro for location of .libs
   * dlopen-test: Add missing libraries to the check list
   * dlopen-test: Move libraries to the right "sections"
   * dlopen-test: Add check for untested libraries
   * BUILD: Fix linking with librt
   * KRB5: Remove spurious warning in logs
   * TESTS: Check new line at end of file
   * UTIL: Fix implicit declaration of function 'htobe32'
   * SYSDB: Remove unused prototype from header file
   * sssctl: Fix missing declaration
   * UTIL: Fix compilation of sss_utf8 with libunistring
   * CONFDB: Supress clang false passitive warnings
   * SIFP: Fix warning format-security
   * RESPONDER: Remove dead assignment to the variable ret
   * Fix compilation with python3.6
   * intg: Generate tmp dir with lowercase
   * LDAP: Fix debug messages after errors in ``*_get_send``
   * LDAP: Removed unused attr_type from users_get_send
   * LDAP: Remove unused parameter attr_type from groups_get_send
   * DP: Remove unused constants BE_ATTR_*
   * DP: Remove unused attr_type from struct dp_id_data
   * LDAP: Remove attrs_type related TODO comments
   * sssd_ldb.py: Remove a leftover debug message
   * intg: Fix python2,3 urllib
   * intg: Avoid using xrange in tests
   * intg: Avoid using iteritems for dictionary
   * intg: Use bytes with hash function
   * intg: Fix creating of slapd configuration
   * intg: Use bytes for value of attributes in ldif
   * intg: Use bytes as input in ctypes
   * intg: Return strings from ctypes wrappers
   * intg: Convert output of executed commands to strings
   * intg: Return list for enumeration functions
   * SYSDB: Update filter for get object by id
   * sysdb-tests: Add test for sysdb_search_object_by_id
   * sysdb: Search also aliases in sysdb_search_object_by_name
   * sysdb-tests: Add test for sysdb_search_object_by_name
   * MONITOR: Fix warning with undefined macro HAVE_SYSTEMD
   * UTIL: Unset O_NONBLOCK for ldap connection

 * Michal Židek (7):

   * sssctl: Flags for command initialization
   * ipa: Nested netgroups do not work
   * common: Fix domain case sensitivity init
   * sssctl: Search by alias
   * sssctl: Case insensitive filters
   * tests: sssctl user/group-show basic tests
   * MAN: sssctl debug level

 * Mike Ely (1):

   * ad_access_filter search for nested groups

 * Pavel Březina (40):

   * cache_req: move from switch to plugins; add logic
   * cache_req: move from switch to plugins, add plugins
   * cache_req: switch to new code
   * cache_req: delete old code
   * sudo: do not store usn if no rules are found
   * nss: move nss_ctx->global_names to rctx
   * ifp: remove unused fields from state
   * setent_notify: remove unused private context
   * sss_crypto.h: include required headers
   * sss_output_name: do not require fq name
   * cache_req: fix initgroups by name
   * cache_req: skip first search on bypass cache
   * cache_req: encapsulate output data into structure
   * cache_req: add ability to gather result from all domains
   * cache_req: add ability to filter domains by enumeration
   * cache_req: add user enumeration
   * cache_req: add group enumeration
   * cache_req: add support for service by name
   * cache_req: add support for service by port
   * cache_req: add support for services enumeration
   * cache_req: add support for netgroups
   * cache_req: allow shallow copy of result
   * cache_req: allow to return well known object as result
   * cache_req: return well known objects in object by sid
   * cache_req: make sure that we always fetch default attrs
   * cache_req: allow upn search with attrs
   * cache_req: add object by name
   * cache_req: add object by id
   * cache_req: make plug-ins definition const
   * cache_req: improve debugging
   * cache_req: fix plugin function description
   * cache_req: allow to search subdomains without fqn
   * cache_req: do not set ncache if dp request fails
   * responders: unify usage of sss_cmd_send_empty and _error
   * responders: remove checks that are handled inside cache_req
   * responders: do not try to contact DP with LOCAL provider
   * utils: add sss_ptr_hash module
   * nss: rewrite nss responder so it uses cache_req
   * nss: make nss responder tests work with new code
   * nss: remove the old code

 * Petr Cech (2):

   * SYSDB: Adding message to inform which cache is used
   * SYSDB: Adding message about reason why cache changed

 * Petr Čech (5):

   * SYSDB: Adding lowercase sudoUser form
   * TESTS: Extending sysdb sudo store tests
   * RESPONDER: Adding of return value checking
   * UTIL: Removing of never read value
   * SYSDB: Fixing of sudorule without a sudoUser

 * Sorah Fukumori (1):

   * BUILD: Fix installation without samba

 * Sumit Bose (11):

   * sysdb: add parent_dom to sysdb_get_direct_parents()
   * sdap: make some nested group related calls public
   * LDAP/AD: resolve domain local groups for remote users
   * PAM: add a test for filter_responses()
   * PAM: add pam_response_filter option
   * IPA/AD: check auth ctx before using it
   * krb5: Use command line arguments instead env vars for krb5_child
   * krb5: fix two memory leaks
   * krb5: add tests for common functions
   * sss_ptr_hash_delete_all: use unsigned long int
   * libwbclient-sssd: wbcLookupSid() allow NULL arguments

 * Victor Tapia (1):

   * MONITOR: Create pidfile after responders started
