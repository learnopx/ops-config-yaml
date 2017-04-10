High level design of OPS-CONFIG-YAML
====================================

The ops-config-yaml library consists of functions to parse and access hardware description file contents, as well as an i2c device access function that is integrated with the hardware description access code.

Responsibilities
----------------
The ops-config-yaml library provides access to the hardware description files defined for a subsystem, giving software convenient methods for accessing content of the hardware-specific information about the switch. It also provides an interface to execute i2c operations to a device. It is the i2c operation function's responsibility to determine additional i2c operations required as a precursor to accessing a device, such as configuring an i2c mux to provide access to the bus segment that the device is connected to.

Design choices
--------------
The OpenSwitch team chose YAML as the format for the hardware description files over other textual representations. Other representations considered include XML and JSON. YAML was selected as it has a relatively simple and uncluttered format that is relatively easy for humans to read and edit.

The ops-config-yaml library relies on the [yaml-cpp][1] library. This yaml parsing library was selected for its powerful Object Oriented API and liberal licensing terms.

Internal Structure
------------------
```ditaa
                     +------------+
                     |Application |
             +-------+-----+------+--------+
             |             |               |
             |             |               |
    +--------v--+    +-----v-----+  +------v-----+
    |i2c_execute|    |yaml_get_* |  |yaml_parse_*|
    +-----------+    |yaml_find_*|  +----+-------+---+
                     +-----+-----+       |           |
                           |             |           |
                     +-----v-------------v---+   +---v----------------------+
                     |Internal Representation|   |Hardware Description Files|
                     +-----------------------+   +--------------------------+

```
There are a number of daemons that use ops-config-yaml to access the hardware description file and execute i2c operations to devices identified in those files. All of these daemons use ops-config-yaml in similar ways.

They create a handle to be used with all other ops-config-yaml library calls by calling *yaml\_new\_config\_handle*, which returns an opaque *YamlConfigHandle* value.

```
typedef void *YamlConfigHandle;

YamlConfigHandle yaml_new_config_handle(void);
```

They add one or more subsystems by calling *yaml\_add\_subsystem*.

```
int yaml_add_subsystem(
    YamlConfigHandle handle,
    const char *subsyst,
    const char *dir_name);
```

They add the relevant hardware description files for the information and access
they need by calling the *yaml\_parse\_\** functions.

```
int yaml_parse_devices(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_parse_thermal(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_parse_ports(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_parse_fans(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_parse_psus(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_parse_leds(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_parse_fru(
    YamlConfigHandle handle,
    const char *subsyst);
```

They request information from the library using *yaml\_get\_\** and *yaml\_find\_\** functions.


Power Supplies
--------------
```
typedef struct {
    int         number_psus;        /* Number of power supplies */
    int         polling_period;     /* Polling period in miliseconds */
} YamlPsuInfo;

typedef struct {
    int         number;
    i2c_bit_op  *psu_present;
    i2c_bit_op  *psu_input_ok;
    i2c_bit_op  *psu_output_ok;
} YamlPsu;

const YamlPsuInfo *yaml_get_psu_info(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_get_psu_count(
    YamlConfigHandle handle,
    const char *susyst);

const YamlPsu *yaml_get_psu(
    YamlConfigHandle handle,
    const char *subsyst,
    unsigned int idx);
```
LEDs
----
```
typedef enum {
    LED_UNKNOWN,
    LED_LOC
} YamlLedTypeValue;

typedef struct {
    unsigned char off;
    unsigned char on;
    unsigned char flashing;
} YamlLedTypeSettings;

typedef struct {
    char        *name;
    char        *type;
    i2c_bit_op  *led_access;
} YamlLed;

typedef struct {
    char                *type;
    YamlLedTypeValue    value;
    YamlLedTypeSettings settings;
} YamlLedType;

typedef struct {
    int number_leds;
    int number_types;
} YamlLedInfo;

const YamlLedInfo *yaml_get_led_info(
    YamlConfigHandle handle,
    const char *subsyst);

int yaml_get_led_count(
    YamlConfigHandle handle,
    const char *subsyst);

const YamlLed *yaml_get_led(
    YamlConfigHandle handle,
    const char *subsyst,
    unsigned int idx);

const YamlLedType *yaml_get_led_type(
    YamlConfigHandle handle,
    const char *subsyst,
    unsigned int idx);
```
Fans
----
```
typedef struct {
    int                 number_fan_frus;    /* Number of Fan FRUs(not fans) */
    int                 fans_per_fru;       /* Number of Fans per FRU */
    YamlFanControlType  fan_speed_control_type; /* YamlFanControlType */
    i2c_bit_op          *fan_speed_control; /* Op to set fan speed */
    YamlFanSpeed        fan_speed_min;      /* Minimum fan speed */
    YamlSpeedSettings   fan_speed_settings; /* Values for each speed  */
    YamlFanDirection    direction;          /* Airflow direction */
    i2c_bit_op          *fan_direction_control; /* dir ctrl op values */
    YamlDirectionValues direction_values;   /* op values to set direction */
    YamlDirectionValues direction_control_values; /* dir ctrl values */
    int                 fan_speed_multiplier; /* multiplier to set speed */
    YamlLedValues       fan_led_values;     /* Fan LED values */
} YamlFanInfo;

typedef struct {
    char        *name;      /* Name identified of the Fan */
    i2c_bit_op  *fan_fault; /* op values for accessing fan fault */
    i2c_bit_op  *fan_speed; /* op values for accessing fan speed */
} YamlFan;

typedef struct {
    int         number;     /* Fan FRU identifier number */
    YamlFan     **fans;     /* YamlFan pointers for fans in the FRU */
    i2c_bit_op  *fan_leds;  /* op values to access the LEDs */
    i2c_bit_op  *fan_direction_detect; /* op values to set fan direction */
} YamlFanFru;

const YamlFanFru *yaml_get_fan_fru(
    YamlConfigHandle handle,
    const char *subsyst,
    unsigned int idx);

int yaml_get_fan_fru_count(
    YamlConfigHandle handle,
    const char *subsyst);

const YamlFanInfo *yaml_get_fan_info(
    YamlConfigHandle handle,
    const char *subsyst);
```
Temperature Sensors
-------------------
```
typedef struct {
    float emergency_on;     /* Min temp to set emergency level */
    float emergency_off;    /* Max temp to remove emergency level */
    float critical_on;      /* Min temp to set critical level */
    float critical_off;     /* Max temp to remove critical level */
    float max_on;           /* Min temp to set fan speed to max */
    float max_off;          /* Max temp to drop max fan speed */
    float min;              /* Max temp to set fan speed to min */
    float low_crit;         /* Max temp to set low critical level */
} YamlAlarmThresholds;

typedef struct {
    float max_on;           /* Min temp to set fan speed to max */
    float max_off;          /* Max temp to drop fan speed below max */
    float fast_on;          /* Min temp to set fan speed to fast */
    float fast_off;         /* Max temp to drop fan speed below fast */
    float medium_on;        /* Min temp to set fan speed to medium */
    float medium_off;       /* Max temp to drop fan speed below medium */
} YamlFanThresholds;

typedef struct {
    int     number_sensors; /* Number of sensors in this subsystem */
    int     polling_period; /* Polling period in miliseconds */
    bool    auto_shutdown;  /* True if auto shutdown for emergency level */
} YamlThermalInfo;

typedef struct {
    int                 number;     /* Sensor identifier */
    char                *location;  /* Sensor location description */
    char                *device;    /* Device name for the sensor */
    char                *type;      /* Sensor type */
    YamlAlarmThresholds alarm_thresholds; /* YamlAlarmThresholds struct */
    YamlFanThresholds   fan_thresholds;   /* YamlFanThresholds struct */
} YamlSensor;

const YamlSensor *yaml_get_sensor(
    YamlConfigHandle handle,
    const char *subsyst,
    unsigned int idx);

int yaml_get_sensor_count(
    YamlConfigHandle handle,
    const char *subsyst);

const YamlThermalInfo *yaml_get_thermal_info(
    YamlConfigHandle handle,
    const char *subsyst);
```
Ports
-----
```
typedef struct {
    int     number_ports;           /* Number of ports in this subsystem */
    int     max_port_speed;         /* Max port speed in Mbits/sec */
    int     max_transmission_unit;  /* Max MTU */
    int     max_lag_count;          /* Max support lags */
    int     max_lag_member_count;   /* Max members allowed in a lag */
    bool    l3_port_requires_internal_vlan; /* L3 port requires internal VLAN */
} YamlPortInfo;

typedef struct {
    i2c_bit_op  *sfpp_tx_disable;   /* i2c op to enable/disable tx */
    i2c_bit_op  *sfpp_tx_fault;     /* i2c op to determine if tx fault */
    i2c_bit_op  *sfpp_rx_loss;      /* i2c op to determine if rx loss */
    i2c_bit_op  *sfpp_mod_present;  /* i2c op to determine module presence */
    i2c_bit_op  *sfpp_interrupt;    /* i2c op for sfp+ interrupt */
} YamlSfpModuleSignals;

typedef struct {
    i2c_bit_op  *qsfpp_reset;       /* i2c op to reset the module */
    i2c_bit_op  *qsfpp_mod_present; /* i2c op to determine module presence */
    i2c_bit_op  *qsfpp_int;         /* interrupt status,
                                        0: generates interrupt when present
                                        1: no interrupt */
    i2c_bit_op  *qsfpp_lp_mode;     /* low power mode,
                                        0: set LPMODE in High Power
                                        1: set LPMODE in Low Power */
    i2c_bit_op  *qsfpp_interrupt;   /* i2c op for qsfp+ interrupt */
} YamlQsfpModuleSignals;

typedef union {
    YamlSfpModuleSignals   sfp;     /* SFP+ op commands */
    YamlQsfpModuleSignals  qsfp;    /* QSFP+ op commands */
} YamlModuleSignals;

typedef struct {
    char              *name;       /* Name identifier for the port */
    bool              pluggable;   /* True if supports pluggable modules */
    char              *connector;  /* Connector type, e.g. SFP, QSFP */
    int               max_speed;   /* Max speed in Mb/sec */
    int               **speeds;    /* List of supports speeds in Mb/sec */
    unsigned int      device;      /* ID of switch port is connected to */
    unsigned int      device_port; /* Dev ID on the switch for this port */
    char              **subports; /* List of subports if port is splittable */
    char              **capabilities; /* Port capabilities */
    char              **supported_modules; /* List of supported modules */
    char              *module_eeprom; /* i2c device ID for module EEPROM */
    char              *parent_port;   /* parent port if subport */
    YamlModuleSignals module_signals; /* i2c ops for this module */
    unsigned int      subport_number; /* sub id of a subport */
} YamlPort;

const YamlPort *yaml_get_port(
    YamlConfigHandle handle,
    const char *subsyst,
    unsigned int idx);

int yaml_get_port_count(
    YamlConfigHandle handle,
    const char *subsyst);

YamlPortInfo *yaml_get_port_info(
    YamlConfigHandle handle,
    const char *subsyst);
```
FRU Info
-----
```
typedef struct {
   char *country_code;
   char device_version;
   char *diag_version;
   char *label_revision;
   char *base_mac_address;
   char *manufacture_date;
   char *manufacturer;
   int num_macs;
   char *onie_version;
   char *part_number;
   char *platform_name;
   char *product_name;
   char *serial_number;
   char *service_tag;
   char *vendor;
} YamlFruInfo;
```
Internal Data Structures
------------------------

Each subsystem added to the configuration is represented by a structure that contains all of the parsed data.
```
typedef struct {
    map<string, YamlDevice> device_map;

    map<string, YamlBus>    bus_map;

    map<string, YamlFile>   file_map;

    YamlSubsysInfo          subsys_info;

    vector<YamlSensor>      sensors;
    YamlThermalInfo         thermal;

    YamlPortInfo            port_info;
    vector<YamlPort>        ports;

    vector<YamlFanFru>      fan_frus;
    YamlFanInfo             fan_info;

    YamlPsuInfo             psu_info;
    vector<YamlPsu>         psus;

    YamlLedInfo             led_info;
    vector<YamlLedType>     led_types;
    vector<YamlLed>         leds;
 
    YamlFruInfo             fru_info;

    vector<i2c_op>          init_ops;

    string                  dir_name;
} YamlSubsystem;
```
The YamlConfigHandle opaque value that the client application uses is actually a pointer to a YamlConfigHandlePrivate structure, which contains a C++ map that allows the code to lookup a YamlSubsystem by its name.
```
typedef struct {
    map<string, YamlSubsystem*> subsystem_map;
} YamlConfigHandlePrivate;
```
References
----------
* [yaml-cpp](https://github.com/jbeder/yaml-cpp "GitHub")
* [yaml.org](http://yaml.org)
* [YAML](https://en.wikipedia.org/wiki/YAML "Wikipedia")

[1]: https://github.com/jbeder/yaml-cpp "GitHub"

