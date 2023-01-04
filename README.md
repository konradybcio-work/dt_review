# Check property order:
Note: Johan said he'd prefer to keep #addr-size cells near reg as they both concern the reg property

* `compatible`
* `reg`
* `reg-names`
* `interrupt-parent`
* `interrupts/interrupts-extended` # mutually exclusive
* `interrupt-names`
* `clocks`
* `clock-names`
* `assigned-clocks`
* `assigned-clock-parents`
* `assigned-clock-rates`
* `operating-points-v2`
* `power-domains`
* `required-opps`
* `phys`

[ ... ]

* `"random" driver-specific props` # reverse-Christmas-tree?
* `status`

### Outliars that need to be decided on:
* `#(clock|reset|power-domain|interrupt|sound-dai|interrupt|[...])-cells`
* `interrupt-controller`

</br>
</br>

# Check if the address part of 'reg' is padded to 8 hex digits
Note: {5,7} instead of {1,7}, as 1-hex-digit-long entries are usually CPUs and these we don't really care about padding, as the reg is just a short fragment of MPIDR_ELn and shorter-than-5-hdl entries are usually some misc subnodes relying on ranges=<> that we never enforced padding on

Note: Bjorn and Krzysztof agreed `reg = <0x0 0xfeedbeef 0x0 0x1000>` is prefered over `reg = <0 0xfeedbeef 0 0x1000>`

* `rg "reg = <(0|0x0) 0x([0-9a-z]{5,7}|[0-9a-z]{9,}) (0|0x0) 0x[0-9a-z]+" arch/arm64/boot/dts/qcom/`
  * Get rid of the `0` case once/if things are unified
* `rg "reg = <0x([0-9a-z]{5,7}|[0-9a-z]{9,}) 0x[0-9a-z]+>" arch/arm64/boot/dts/qcom/ arch/arm/boot/dts/qcom*.dts*`
  * No qcom arm32 socs use, or care about >32bit VAs, for now at least

</br>
</br>

# Check pin definitions
* Ensure there's no unnecessary wrapping of single-pin descriptions into a parent-child style node

* node names need to end in -state/-pins for parent/child

* Order of properties:
  * `pins`
  * `function`
  * `power-source` # qcom spmi gpios
  * `drive-strength`
  * `bias-`
  * `(in|out)put-` # etc

</br>
</br>

# General things
* No double/weird newlines
* No double spaces (except for indentation)
* No mixed spaces/tabs, checkpatch points that out but yknow..
  * Maybe just run checkpatch hmm..
* Indentation has to be proper
* Make sure ARRAY_SIZE(x-names) == ARRAY_SIZE(x)
* Make sure chassis-type is there
* Line length:
  * 80 preferred
  * up to 100 no problemo
  * more only if REALLY REALLY REALLY necessary (very rare)

</br>
</br>

# Node style

Example nodes:

```
#address-cells = <2>; # 2*32 = 64-bit addressing
#size-cells = <2>; # 2*32 = 64-bit sizing

label: name@unitaddress {
        compatible = "foo,bar";
        reg = <ADDR_HI ADDR_LO SIZE_HI SIZE_LO>;
        status = "disabled";
};

e.g.:

/* The definition in a common DTSI */
pm1337_l1: regulator@1feedbeef {
        compatible = "imaginary,pmic-regulator";
        reg = <0x1 0xfeedbeef 0x0 0x1000>;
};

/* Device-specific override */
&pm1337_l1 {
        regulator-min-microvolts = <1000000>;
        regulator-max-microvolts = <1000000>;
};
```

* [Node names should be generic](https://devicetree-specification.readthedocs.io/en/latest/chapter2-devicetree-basics.html#generic-names-recommendation)
* Ensure `status = "okay"` is there only when necessary (e.g. not when we're first defining the node)
  * `"okay"` is idiomatic, don't use `"ok"`
* Node name should use hyphens, not underscores
* Labels should use underscores, not anything else (e.g. hyphens trigger syntax errors)
* Nodes should be overriden using &label references, this is the least error-prone approach
   ```
   /* Common DTSI */
   nice_device: nice@1337babe {
           compatible = "very,nice";
           reg = <0x0 0x1337babe 0x0 0x10>;
   };
   
   /* Device DTS */
   &nice_device {
           compatible = "very,nice-revised";
   };
   ```
  * NOT by defining the node again - very error prone and unmaintainable
   ```
   WRONG, DON'T FOLLOW
   /* Common DTSI */
   somenode@addr {
           compatible = "some,device";
           reg = <0x0 0x0000addr 0x0 0x1000>;
   };
   
   /* Device DTS */
   somenode@addr {
           comaptible = "some,device-v2";  
   };
   ```
  * NOT by referencing the full path - extremely unmaintainable
   ```
   WRONG, DON'T FOLLOW
   somedevice@addr {
           [...]
           interrupt-parent = < &{/soc/interrupt-controller@40000} >;
   };
   ```

</br>
</br>


# Check node sorting:
* Non-MMIO nodes should be sorted alphabetically
* MMIO nodes should be inside /soc(""|@0)
* MMIO nodes should be sorted by address
* All this applies to all subnodes of all nodes, of course

</br>
</br>

# Check misc sorting
* #includes should be sorted alphabetically
 * SoC DTSI has to be included before PMICs for obvious reasons
* dt-bindings includes should be sorted separately from DTSI includes
* check that the thing even compiles..
  * handle dependencies (recursively) based on [1] ([1]: ??) https://... in cover letters or under --- in commit messages?

