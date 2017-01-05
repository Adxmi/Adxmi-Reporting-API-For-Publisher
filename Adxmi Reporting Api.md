# INTRODUCTION
__Adxmi Reporting Api__ provides our publisher a programmatic way to get statistics data. In this document, we will illustrate how to get campaign performance data with Adxmi Reporting Api.

Here is the general principle of Adxmi Reporting Api
1. The Api is only accessible by HTTP GET request and returns the data in JSON.
2. The Api reports the data with following aspects
    * Number of impression
    * Number of click
    * Number of conversion
    * Number of revenue
3. The Api is supported only for the publishing partners who have Adxmi account and approved application.

# Api Request
## Adxmi Report Api url
> http://reporting.yyapi.net/v1/data

## Request Parameter
| Parameter  | Description                                                                      | Type                                                 | Mandatory |
|------------|----------------------------------------------------------------------------------|------------------------------------------------------|-----------|
| app_id     | Apply from www.adxmi.com for your application                                    | string                                               | Y         |
| start_date | The date you wish to start with                                                  | date (yyyy-mm-dd)                                    | Y         |
| end_date   | The date you wish to end with                                                    | date (yyyy-mm-dd)                                    | Y         |
| sign       | Signature for query parameters. See [Signature Algorithm](#sign-algo) for detail | string                                               | Y         |
| dimension  | The way to arrange data                                                          | enum `date,offer,country`                            | N         |
| product    | filter data for specific product                                                 | enum `wall,video,custom,interstitial,customplus,api` | N         |

### Notice
* __dimension__ will give tacit consent to “date” if not set.
* Data for all products will be returned if __product__ is not set.

### Example
http://reporting.yyapi.net/v1/data?appid=93ffeb94fd876e87&start_date=2015-12-05&end_date=2015-12-14&dimension=date&product=custom&sign=e07dc14f319c1dc3da855479804db48b

## Response Field
### Response Parameter by date dimension
| Parameter | Description | Type |
| ---- | ---- | ---- |
| date | yyyy-mm-dd | date |
| impression | Number of impression on the specified date | int |
| click | Number of click on the specified date | int |
| conversion | Number of the conversion on the specified date | int |
| revenue | Number of the total revenue on the specified date | float |

__Example__
```json
{
    c: 0,
    data: [
        {
            date: "2015-12-10",
            impression: 791,
            click: 523,
            conversion: 100,
            revenue: 11.62
        }
}
```

### Response Parameter by offer dimension
| Parameter | Description | Type |
| ---- | ---- | ---- |
| id | Offer id | string |
| name | Offer name | string |
| countries | Targeting countries of offer | array |
| os | The value is android or ios | string |
| payout | The revenue of the offer at the time request | float |
| date | yyyy-mm-dd | date |
| impression | Number of impression for the offer on the specified date | int |
| click | Number of click for the offer on the specified date | int |
| conversion | Number of the conversion for offer on the specified date | int |
| revenue | Number of the total revenue for offer on the specified date | float |

__Example__
```json
{
    c: 0,
    data: [
        {
            id: "730294650560057344",
            name: "DoubleDown Casino - FREE Slots",
            countries: [
                "CA",
                "US"
            ],
            os: [
                "android"
            ],
            payout: 1.2,
            date: "2015-12-11",
            impression: 1220,
            click: 61,
            conversion: 4,
            revenue: 4.8
        }
}
```

### Response Parameter by country dimension
| Parameter | Description | Type |
| ---- | ---- | ---- |
| country | two-letter country code for ISO 3166 | string |
| date | yyyy-mm-dd | date |
| impression | Number of impression in the specified country | int |
| click | Number of click in the specified country | int |
| conversion | Number of the conversion in the specified country | int |
| revenue | Number of the total revenue in the specified country | float |

__Example__
```json
{
   c: 0,
    data: [
        {
            country: "US",
            date: "2015-12-09",
            impression: 7,
            click: 3,
            conversion: 1,
            revenue: 1.46
        }
    ]
}
```

### Error Response
An error response message will be displayed if error occurs.
```json
{
   c: -1,
   msg:”message”
}
```
“message” is the reason why error occurs

# <a name="sign-algo"></a> Signature Algorithm

Use all parameters except `sign` in __Requesting Parameters__ list as key to compute MD5 value. Assume that the parameters participate in computing signature are k1,k2,k3, then the signature calculation method is below:
Formatting the non-binary type request parameters to key=value format. For example: k1=v1,k2=v2,k3=v3 Sort the key-value in alphabet ascending order and connect them together. For example: k1=v1k2=v2k3=v3 Append `app_secret` after the connected key-value string. MD5 of the above string is the signature value.

__Notice__
* Don't include the sign(signature) parameters when calculating the signature.
* The parameters in signature procedure have not been processed by urlencode.
* In order to ensure the signature won't be abnormal when changing the request parameters, please make sure using the verification function we offer as the verification method.
* If developer's request url contains parameters not mentioned above, these parameters will also join in the signature.

##  Signature Function
__For PHP__
```php
function signUrl($url, $app_secret)
{
    $params = array();
    $url_parse = parse_url($url);
    if (isset($url_parse['query'])) {
        $query_arr = explode('&', $url_parse['query']);
        if (!empty($query_arr)) {
            foreach ($query_arr as $p) {
                if (strpos($p, '=') !== false) {
                    list($k, $v) = explode('=', $p);
                    $params[$k] = urldecode($v);
                }
            }
        }
    }

    $before_md5 = '';
    ksort($params);
    foreach ($params as $k => $v) {
        $before_md5 .= "{$k}={$v}";
    }
    $before_md5 .= $app_secret;

    $sign = md5($before_md5);
    return "{$url}&sign={$sign}";
}
```

__For Java__
```java
 import java.io.IOException;
import java.net.MalformedURLException;
import java.net.URL;
import java.net.URLDecoder;
import java.security.GeneralSecurityException;
import java.security.MessageDigest;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.TreeMap;

public class AdxmiSign {
	/**
	 * Signature Generation Algorithm
	 *
	 * @param HashMap
	 *            <String,String> params Request paramenters set, all parameters
	 *            need to convert to string type before this
	 * @param String
	 *            appSecret The secret key for your application
	 * @return String
	 * @throws IOException
	 */
	public static String getSignature(HashMap<String, String> params,
			String appSecret) throws IOException {
		// Sort the parameters in ascending order
		Map<String, String> sortedParams = new TreeMap<String, String>(params);

		Set<Map.Entry<String, String>> entrys = sortedParams.entrySet();
		// Traverse the set after sorting, connect all the parameters as
		// "key=value" format
		StringBuilder basestring = new StringBuilder();
		for (Map.Entry<String, String> param : entrys) {
			basestring.append(param.getKey()).append("=")
					.append(param.getValue());
		}
		basestring.append(appSecret);
		// System.out.println(basestring.toString());
		// Calculate signature using MD5
		byte[] bytes = null;
		try {
			MessageDigest md5 = MessageDigest.getInstance("MD5");
			bytes = md5.digest(basestring.toString().getBytes("UTF-8"));
		} catch (GeneralSecurityException ex) {
			throw new IOException(ex);
		}
		// Convert the MD5 output binary result to lowercase hexadecimal result.
		StringBuilder sign = new StringBuilder();
		for (int i = 0; i < bytes.length; i++) {
			String hex = Integer.toHexString(bytes[i] & 0xFF);
			if (hex.length() == 1) {
				sign.append("0");
			}
			sign.append(hex);
		}
		return sign.toString();
	}

	/**
	 * Calculate signature with a completed unsigned URL, append the signature
	 * at the end of this URL.
	 *
	 * @param String
	 *            url The URL to be signed
	 * @param String
	 *            appSecret The secret key for your application
	 * @return String
	 * @throws IOException
	 *             , MalformedURLException
	 */
	public static String signUrl(String url, String appSecret) throws Exception {
		try {
			URL urlObj = new URL(url);
			String query = urlObj.getQuery();
			String[] params = query.split("&");
			Map<String, String> map = new HashMap<String, String>();
			for (String each : params) {
				String name = each.split("=")[0];
				String value;
				try {
					value = URLDecoder.decode(each.split("=")[1], "UTF-8");
				} catch (Exception e) {
					value = "";
				}
				map.put(name, value);
			}
			String signature = getSignature((HashMap<String, String>) map,
					appSecret);
			return url + "&sign=" + signature;
		} catch (MalformedURLException e) {
			throw e;
		} catch (IOException e) {
			throw e;
		}
	}
}
```

__For Python__
```python
from urlparse import urlparse
from urllib import unquote_plus
from hashlib import md5

def signUrl(url, app_secret):
    params = {}
    url_parse = urlparse(url)
    query = url_parse.query
    query_array = query.split('&')

    for group in query_array:
        k, v = group.split('=')
        params[k] = unquote_plus(v)
    del k, v

    sorted_params = sorted(params.items(), key=lambda d: d[0])

    before_md5 = ''.join(['%s=%s' % (k, v) for k, v in sorted_params])
    before_md5 += app_secret

    m = md5()
    m.update(before_md5)
    return'%s&sign=%s' % (url, m.hexdigest())
```

