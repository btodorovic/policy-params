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

