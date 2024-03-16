# Simple Implementation of Junos Policies with Parameters

This demo shows an equivalent of the Cisco IOS-XR route-policy wth arguments feature - constructs like:

**IOS-XR:**
	route-policy RP-EBGP-EXPORT ($tag)
	    if destionation in PFX-$tag then
	        set community CM-$tag additive
	    else
	        drop
	    endif
	end-policy

	route-policy RP-EBGP-EXPORT-1234
	    apply RP-EBGP-EXPORT(1234)
	endif

	route-policy RP-EBGP-EXPORT-5678
	    apply RP-EBGP-EXPORT(5678)
	endif

Plain Junos OS CLI does not provide an equivalent feature. However, you can mimic this behavior using the so-called [Commit Script Macros](https://www.juniper.net/documentation/us/en/software/junos/automation-scripting/topics/example/junos-script-automation-commit-script-creating-custom-syntax-with-macros.html) feature.

In the example below we assume that the "template" policy should look like this:

**Junos OS - TEMPLATE (not applicable directly to the Router CLI!):**

	policy-options {
	    policy-statement PS-EBGP-EXPORT-$tag {
	        term ACCEPT-$tag {
	            from route-filter-list RFL-$tag;
	            then {
	                community add CM-$tag;
	                accept;
	            }
	        }
	        then reject;
	    }
	}

Regrettably, we cannot apply this template directly into the router CLI, but we can do the following:

* Create a commit script containing a template function with variables
* Create a macro containing the list of variable values

Let's start from the latter step - in the example above we will use the argument **$name = 'PS-EBGP-EXPORT'** and a list of **$tag** values associated with that name:

**Junos OS - macro definition:**
	policy-options {
	    apply-macro MC-EBGP-EXPORT {
	        1201;
	        1202;
	        1204;
	        1207;
	        1213;
	        name PS-EBGP-EXPORT;
	    }
	}

Also, create route-filter-lists manually:

<pre>
policy-options {
    route-filter-list RFL-1201 {
        15.0.0.0/8 orlonger;
    }
    route-filter-list RFL-1202 {
        17.0.0.0/8 orlonger;
        18.2.0.0/16 orlonger;
    }
    route-filter-list RFL-1204 {
        15.0.0.0/8 orlonger;
    }
    route-filter-list RFL-1207 {
        15.0.0.0/8 orlonger;
    }
    route-filter-list RFL-1213 {
        15.0.0.0/8 orlonger;
    }
}
</pre>

In the next step - create and install the commit script:

**Install commit script:**
	set system scdripts commit allow-transients file policy-params.slax

The script contents can be seen [here](./policy-params.slax).

When commited, this expands into the **ephemeral** configuration, which cannot be seen directly (you need to use **show configuration | display commit-scripts** to see it):

<pre>
policy-options {
    apply-macro MC-EBGP-EXPORT {
        1201;
        1202;
        1204;
        1207;
        1213;
        name PS-EBGP-EXPORT;
    }
    route-filter-list RFL-1201 {
        15.0.0.0/8 orlonger;
    }
    route-filter-list RFL-1202 {
        17.0.0.0/8 orlonger;
        18.2.0.0/16 orlonger;
    }
    route-filter-list RFL-1204 {
        15.0.0.0/8 orlonger;
    }
    route-filter-list RFL-1207 {
        15.0.0.0/8 orlonger;
    }
    route-filter-list RFL-1213 {
        15.0.0.0/8 orlonger;
    }
    policy-statement PS-EBGP-EXPORT-1201 {
        term ACCEPT-1201 {
            from {
                route-filter-list RFL-1201;
            }
            then {
                community add CM-1201;
                accept;
            }
        }
        then reject;
    }
    policy-statement PS-EBGP-EXPORT-1202 {
        term ACCEPT-1202 {
            from {
                route-filter-list RFL-1202;
            }
            then {
                community add CM-1202;
                accept;
            }
        }
        then reject;
    }
    policy-statement PS-EBGP-EXPORT-1204 {
        term ACCEPT-1204 {
            from {
                route-filter-list RFL-1204;
            }
            then {
                community add CM-1204;
                accept;
            }
        }
        then reject;
    }
    policy-statement PS-EBGP-EXPORT-1207 {
        term ACCEPT-1207 {
            from {
                route-filter-list RFL-1207;
            }
            then {
                community add CM-1207;
                accept;
            }
        }
        then reject;
    }
    policy-statement PS-EBGP-EXPORT-1213 {
        term ACCEPT-1213 {
            from {
                route-filter-list RFL-1213;
            }
            then {
                community add CM-1213;
                accept;
            }
        }
        then reject;
    }
    community CM-1201 members 65000:1201;
    community CM-1202 members 65000:1202;
    community CM-1204 members 65000:1204;
    community CM-1207 members 65000:1207;
    community CM-1213 members 65000:1213;
}
</pre>

## Tips for Developing the Script

The easiest way to create the template is to create a prototype policy manually first. For instance:

<pre>
policy-options {
    policy-statement PS-EBGP-EXPORT-1201 {
        term ACCEPT-1201 {
            from {
                route-filter-list RFL-1201;
            }
            then {
                community add CM-1201;
                accept;
            }
        }
        then reject;
    }
    route-filter-list RFL-1201 15.0.0.0/8 orlonger;
    community CM-1201 members 65000:1201;
}
</pre>

To impleement this, you will probably use the standard Junos CLI set-commands - i.e.:

<pre>
set policy-options route-filter-list RFL-1201 15.0.0.0/8 orlonger
set policy-options policy-statement PS-EBGP-EXPORT-1201 term ACCEPT-1201 from route-filter-list RFL-1201
set policy-options policy-statement PS-EBGP-EXPORT-1201 term ACCEPT-1201 then community add CM-1201
set policy-options policy-statement PS-EBGP-EXPORT-1201 term ACCEPT-1201 then accept
set policy-options policy-statement PS-EBGP-EXPORT-1201 then reject
set policy-options community CM-1201 members 65000:1201
</pre>

**DO NOT COMMIT** - rather, obtain the XML format of the policy-statement using: **show policy-options | display xml**, which gives:

<pre>
<rpc-reply xmlns:junos="http://xml.juniper.net/junos/24.1R1.2/junos">
    <configuration junos:changed-seconds="1710597605" junos:changed-localtime="2024-03-16 15:00:05 CET">
            <policy-options>
                <route-filter-list>
                    <name>RFL-1201</name>
                    <rf_list>
                        <address>15.0.0.0/8</address>
                        <orlonger/>
                    </rf_list>
                </route-filter-list>
                <policy-statement>
                    <name>PS-EBGP-EXPORT-1201</name>
                    <term>
                        <name>ACCEPT-1201</name>
                        <from>
                            <route-filter-list>
                                <name>RFL-1201</name>
                            </route-filter-list>
                        </from>
                        <then>
                            <community>
                                <add/>
                                <community-name>CM-1201</community-name>
                            </community>
                            <accept/>
                        </then>
                    </term>
                    <then>
                        <reject/>
                    </then>
                </policy-statement>
                <community>
                    <name>CM-1201</name>
                    <members>65000:1201</members>
                </community>
            </policy-options>
    </configuration>
    <cli>
        <banner>[edit]</banner>
    </cli>
</rpc-reply>
</pre>

Based on the template above, you can create the script:

<pre>
/* === Standard header === */
version 1.0;
ns junos = "http://xml.juniper.net/junos/*/junos";
ns xnm = "http://xml.juniper.net/xnm/1.1/xnm";
ns jcs = "http://xml.juniper.net/junos/commit-scripts/1.0";
import "../import/junos.xsl";
 
match configuration {
    var $root = policy-options;
    for-each ($root/apply-macro[data/name = 'name']) {
        var $policy_name = data[name = 'name']/value;      ### Policy name obtained here

        /* --- Template for policies named PS-EBGP-IMPORT-nnnnn are defined here -- */

        if ($policy_name == 'PS-EBGP-EXPORT') {
            <transient-change> {
                <policy-options> {
                    for-each (data[not(value)]/name) {
                        <policy-statement> {
                            <name>$policy_name _ '-' _ .;
                            <term> {
                                <name>'ACCEPT-' _ .;
                                <from> {
                                    <route-filter-list> {
                                        <name>'RFL-' _ .;
                                    }
                                }
                                <then> {
                                    <community> {
                                        <add>;
                                        <community-name> 'CM-' _ .;
                                    }
                                    <accept>;
                                }   
                            }
                            <then> {
                                <reject>;
                            }
                        }
                        <community> {
                            <name> 'CM-' _ .;
                            <members>'65000:' _ .;
                        }
                    }
                }
            }
        }

        /* --- Template for policies named PS-EBGP-IMPORT-nnnnn are defined here -- */

        else if ($policy_name == 'PS-EBGP-IMPORT') {
            /*
             * --- Template code goes here ... etc.
             */
        }

        /* --- Add similar templates here, as required --- */
    }
}
</pre>
