# basic data models

* leaf
    ```yang
    leaf host-name {
         type string;
         description "Hostname for this system";
    }
    ```
    ```xml
     <host-name>my.example.com</host-name>
    ```
* leaf-list
    ```yang
    leaf-list domain-search {
         type string;
         description "List of domain names to search";
     }
    ```
    ```xml
     <domain-search>high.example.com</domain-search>
     <domain-search>low.example.com</domain-search>
     <domain-search>everywhere.example.com</domain-search>
    ```
* container
    ```yang
    container system {
        container login {
            leaf message {
                type string;
                description
                    "Message given at start of login session";
            }
        }
    }
    ```
    ```xml
    <system>
       <login>
         <message>Good morning</message>
       </login>
    </system>
    ```
* list-nodes
    ```yang
    list user {
        key "name";
        leaf name {
            type string;
        }
        leaf full-name {
            type string;
        }
        leaf class {
            type string;
        }
    }
    ```
    ```xml
    <user>
      <name>glocks</name>
      <full-name>Goldie Locks</full-name>
      <class>intruder</class>
    </user>
    <user>
      <name>snowey</name>
      <full-name>Snow White</full-name>
      <class>free-loader</class>
    </user>
    <user>
      <name>rzell</name>
      <full-name>Rapun Zell</full-name>
      <class>tower</class>
    </user>
    ```
* module
    ```yang
    module acme-system {
        namespace "http://acme.example.com/system";
        prefix "acme";

        organization "ACME Inc.";
        contact "joe@acme.example.com";
        description
            "The module for entities implementing the ACME system.";

        revision 2007-06-09 {
            description "Initial revision.";
        }

        container system {
        ...
        }
    }
    ```
* state-data
    * has `config false` in its definition
    * will be return in `<get>` and not in `<get-config>` operation
        ```yang
            list interface {
                key "name";

                leaf name {
                    type string;
                }
                leaf speed {
                    type enumeration {
                        enum 10m;
                        enum 100m;
                        enum auto;
                    }
                }
                leaf observed-speed {
                    type uint32;
                    config false;      // <--
                }
            }
        ```

# built in types

       +---------------------+-------------------------------------+
       | Name                | Description                         |
       +---------------------+-------------------------------------+
       | binary              | Any binary data                     |
       | bits                | A set of bits or flags              |
       | boolean             | "true" or "false"                   |
       | decimal64           | 64-bit signed decimal number        |
       | empty               | A leaf that does not have any value |
       | enumeration         | Enumerated strings                  |
       | identityref         | A reference to an abstract identity |
       | instance-identifier | References a data tree node         |
       | int8                | 8-bit signed integer                |
       | int16               | 16-bit signed integer               |
       | int32               | 32-bit signed integer               |
       | int64               | 64-bit signed integer               |
       | leafref             | A reference to a leaf instance      |
       | string              | Human-readable string               |
       | uint8               | 8-bit unsigned integer              |
       | uint16              | 16-bit unsigned integer             |
       | uint32              | 32-bit unsigned integer             |
       | uint64              | 64-bit unsigned integer             |
       | union               | Choice of member types              |
       +---------------------+-------------------------------------+

# derived types

* typedef
* reusable
    * grouping
    * refines
* choices
    ```yang
     container food {
       choice snack {
           case sports-arena {
               leaf pretzel {
                   type empty;
               }
               leaf beer {
                   type empty;
               }
           }
           case late-night {
               leaf chocolate {
                   type enumeration {
                       enum dark;
                       enum milk;
                       enum first-available;
                   }
               }
           }
       }
    }
    ```
    ```xml
    <food>
      <pretzel/>
      <beer/>
    </food>
    ```
* augment
    ```yang
    augment /system/login/user {
        when "class != 'wheel'";
        leaf uid {
            type uint16 {
                range "1000 .. 30000";
            }
        }
    }
    ```

# rpc definition

```yang
     rpc activate-software-image {
         input {
             leaf image-name {
                 type string;
             }
         }
         output {
             leaf status {
                 type string;
             }
         }
     }

```
```xml
     <rpc message-id="101"
          xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
       <activate-software-image xmlns="http://acme.example.com/system">
         <image-name>acmefw-2.3</image-name>
      </activate-software-image>
     </rpc>

     <rpc-reply message-id="101"
                xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
       <status xmlns="http://acme.example.com/system">
         The image acmefw-2.3 is being installed.
       </status>
     </rpc-reply>

```

## notification

```yang
     notification link-failure {
         description "A link failure has been detected";
         leaf if-name {
             type leafref {
                 path "/interface/name";
             }
         }
         leaf if-admin-status {
             type admin-status;
         }
         leaf if-oper-status {
             type oper-status;
         }
     }
```
```xml
     <notification
         xmlns="urn:ietf:params:netconf:capability:notification:1.0">
       <eventTime>2007-09-01T10:00:00Z</eventTime>
       <link-failure xmlns="http://acme.example.com/system">
         <if-name>so-1/2/3.0</if-name>
         <if-admin-status>up</if-admin-status>
         <if-oper-status>down</if-oper-status>
       </link-failure>
     </notification>
```

# Language concepts

## module

* The module is the base unit of definition in YANG. 
* defines a single data model
* A module can define a complete, cohesive model,
  or augment an existing data model with additional nodes.

# netconf

## operations

```
<get>           | Retrieve running configuration and device state information
<get-config>    | Retrieve all or part of a specified configuration datastore
<edit-config>   | Edit a configuration datastore by creating, deleting, merging or replacing content
<copy-config>   | Copy an entire configuration datastore to another configuration datastore
<delete-config> | Delete a configuration datastore
<lock>          | Lock an entire configuration datastore of a device
<unlock>        | Release a configuration datastore lock previously obtained with the <lock> operation
<close-session> | Request graceful termination of a NETCONF session
<kill-session>  | Force the termination of a NETCONF session
```
