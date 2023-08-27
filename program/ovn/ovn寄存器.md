* metadata
代表逻辑数据通路(logical datapath)

* register number 10
The logical flags are intended to handle keeping context between tables in order to decide which rules in subsequent tables are atched
* register number 11
OVN stores the zone information for north to south traffic (for DNTting or ECMP symmetric replies)
* register number 12
OVN stores the zone information for south to north traffic (for SNATing)
* register number 13
conntrack zone field for logical ports
* register number 14
logical input port
* re