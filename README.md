This page will contain notes I took while working with snmp4j library and implementing SNMP trap emitter and trap listener with the SNMP v3 support.  

## Converting Date and Time to DateAndTime Octet String  

Per [rfc 2579](https://tools.ietf.org/html/rfc2579) DateAndTime data type is represented as Octet String  

![image](https://user-images.githubusercontent.com/37493923/109395075-12555b00-78f0-11eb-8737-58ce0dca7b70.png)

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
            DateAndTimeInHEX = year + month + day + hour + minute + second + millisecond + hexoffset + houroffset + minuteoffset;
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
