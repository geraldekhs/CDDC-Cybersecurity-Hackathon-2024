# Crypto 2 (50 pts)

#### Decrypt the given string: FGGF24{FK3FN_0XW_WK3_Q3Z_P0Y13_DW_8SP}
* This is a basic substitution cipher with a shift of 3 letters from C to F.
* Reversing the shift returns the decoded message:

    ```{.python .cb-nb}
    string = 'FGGF24{FK3FN_0XW_WK3_Q3Z_P0Y13_DW_8SP}'
    string2 = ''

    for i in string:
        if i.isalpha():
            string2+=chr(ord(i)-3)
        else:
            string2+=i

    print(string2)
    ```

# OSINT 1 (250 pts)

#### An random string is given. Find the flag from it. Some other clues are given eg. flag{landmark} 
* Analysis by Gemini 1.5 Pro reveals that the string is a IFPS content identifier (CID).

**IFPS Gateway Analysis**
* Select a random hostname using https://ipfs.github.io/public-gateway-checker/.
* Replace the content after the last slash in the hostname with the given IFPS CID eg:

    https://cloudflare-ipfs.com/ipfs/bafybeifx7yeb55armcsxwwitkymga5xf53dxiarykms3ygqic223w5sk3m
* The modified URL can now be used to access the IPFS content on the link, which is a file containing 1000 photos of money bags.

**Photo Analysis**
* https://www.aperisolve.com/ was used to check for any information embedded in the images.
* GPS data was found to be present in `exiftool` tab.
* Thus, a local `exiftool` command was used to extract GPS locations of all images in the directory `moneys` and dump them into a text file for further analysis:
    
    `exiftool -location:all -G -a -s moneys > locations.txt`

**Further Analysis of GPS Locations**
* Dumped GPS locations in the text file are shown below.

    ![image-2.png](attachment:image-2.png)

* The composite GPS position was extracted and converted to the format: '[number]N, [number]E', with S and W converted to their counterparts using a negative sign eg. 40S, 35W --> -40N, -35E using the function below:

    ```{.python .cb-nb}
    arr = []
    with open('locations.txt', 'r') as file:
        for line in file:
            if 'GPSPosition' in line:
                # Split the line by ':' and remove leading/trailing whitespace
                parts = line.split(':')[1].strip()
                # Split the coordinates into latitude and longitude
                latitude, longitude = parts.split(',') 
                # Convert to decimal degrees

                # print(latitude)
                def convert_to_decimal(coordinate_str):
                    parts = coordinate_str.split(" ")
                    # print(parts)
                    degrees = parts[1].replace('deg', '')
                    minutes = float(parts[2].replace("'", "")) / 60
                    seconds = float(parts[3].replace("\"", "")) / 3600

                    return float(parts[0]) + minutes + seconds

                lat_decimal = convert_to_decimal(latitude)
                lon_decimal = convert_to_decimal(longitude.strip())
                
                # Format the coordinates as "44.150928N, 171.471131W"
                formatted_coords = f"{lat_decimal:.6f}{latitude.split()[-1]}, {lon_decimal:.6f}{longitude.split()[-1]}"
                arr.append(formatted_coords)
    print(arr)
    ```