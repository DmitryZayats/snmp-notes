This page contains notes I took while working with [snmp4j](http://www.snmp4j.com/) library and implementing SNMP trap emitter and trap listener with the SNMP v3 support.  

## Links  

- [SNMP4J Main page](http://www.snmp4j.com/)  
- [SNMP4J Downloads, License, API Doc, FAQ, Forum links](https://agentpp.com/api/java/snmp4j.html)  
- [Download SNMP4J](https://agentpp.com/download.html#SNMP4J)  
- [Java API: Package org.snmp4j documentation. Has some useful examples.](https://agentpp.com/doc/snmp4j/org/snmp4j/package-summary.html)  
- [Java API: Package org.snmp4j.security. Needed for implementing v3.](https://agentpp.com/doc/snmp4j/org/snmp4j/security/package-summary.html)
- [Java API: Package org.snmp4j.smi. SMI Data Types](https://agentpp.com/doc/snmp4j/org/snmp4j/smi/package-summary.html)

## Converting Date and Time to DateAndTime Octet String  

Per [rfc 2579](https://tools.ietf.org/html/rfc2579) DateAndTime data type is represented as Octet String  

```
DateAndTime ::= TEXTUAL-CONVENTION
    DISPLAY-HINT "2d-1d-1d,1d:1d:1d.1d,1a1d:1d"
    STATUS       current
    DESCRIPTION
            "A date-time specification.

            field  octets  contents                  range
            -----  ------  --------                  -----
              1      1-2   year*                     0..65536
              2       3    month                     1..12
              3       4    day                       1..31
              4       5    hour                      0..23
              5       6    minutes                   0..59
              6       7    seconds                   0..60
                           (use 60 for leap-second)
              7       8    deci-seconds              0..9
              8       9    direction from UTC        '+' / '-'
              9      10    hours from UTC*           0..13
             10      11    minutes from UTC          0..59

            * Notes:
            - the value of year is in network-byte order
            - daylight saving time in New Zealand is +13

            For example, Tuesday May 26, 1992 at 1:30:15 PM EDT would be
            displayed as:

                             1992-5-26,13:30:15.0,-4:0

            Note that if only local time is known, then timezone
            information (fields 8-10) is not present."
    SYNTAX       OCTET STRING (SIZE (8 | 11))
```  

Below method will convert human readable Date And Time String to HEX Octet String that can be used when building SNMP traps.  

```.java
    public static String DateTimeToOctetString(String DateAndTime){

        String DateAndTimeInHEX;
        // Example DateAndTime string that can be parsed by below regexp 2020-12-3,14:9:24.0,+0:0
        Pattern p = Pattern.compile("(\\d{4}?)-(\\d+)-(\\d+),(\\d+):(\\d+):(\\d+)\\.(\\d+),([+-])(\\d+):(\\d+)");
        Matcher m = p.matcher(DateAndTime);

        if(m.find()){
            String year=String.format("%04x",Integer.parseInt(m.group(1)));
            String month=String.format("%02x",Integer.parseInt(m.group(2)));
            String day=String.format("%02x",Integer.parseInt(m.group(3)));
            String hour=String.format("%02x",Integer.parseInt(m.group(4)));
            String minute=String.format("%02x",Integer.parseInt(m.group(5)));
            String second=String.format("%02x",Integer.parseInt(m.group(6)));
            String millisecond=String.format("%02x",Integer.parseInt(m.group(7)));
            String hexoffset = Integer.toHexString(m.group(8).toCharArray()[0]);
            String houroffset=String.format("%02x",Integer.parseInt(m.group(9)));
            String minuteoffset=String.format("%02x",Integer.parseInt(m.group(10)));
            DateAndTimeInHEX = year + month + day + hour + minute + second + 
            millisecond + hexoffset + houroffset + minuteoffset;
        }
        else {
            throw new RuntimeException("Date And Time is not properly formatted");
        }
        return DateAndTimeInHEX;
    }
```  

We need to pass to this method properly formatted Date And Time String so that regexp can parse it.  
Format for setting arbitrary Date and Time is  
Year-Month-Day,Hour:Minute:Second.millisecond,{+ or - time zone offset}{hour offset}:{minute offset}  
Example: 2020-12-5,14:9:24.0,+0:0  

Example of creating Date And Time String in the proper format can be done like in the example below  

```.java
    public static String DateAndTimeNow(){
        ZonedDateTime date = ZonedDateTime.now();
        DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd,HH:m:s.S,xxx");
        String dateAndTime = date.format(formatter);
        return dateAndTime;
    }
```  
