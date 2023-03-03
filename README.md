``` 
const permLocale = "permanentLocaleCookie";
const userLocale = "userSelectedLocaleCoookie";

const countryMap = {  
  CA: {
    default: "ca/en/",
    QC: "ca/fr/"
  },
  DE: {
    default: "de/de/"
  },
  FR: {
    default: "fr/fr/"
  },
  IN: {
    default: "in/en/"
  },
  US: {
    default: "us/en/"
  },
  UK: {
    default: "uk/en/"
  }
};

export default {
  async fetch(request) {

    const url = new URL(request.url);
    console.log(url.pathname)

    // Only redirect root to avoid loops
    // if (url.pathname == "/") {
      console.log("Root path, checking location")
      let locale

      // Check for user specified locale; If no user specified locale check for permanent locale
      if (getCookie(request, userLocale)) {
        locale = getCookie(request, userLocale)
      }
      else if (getCookie(request, permLocale)) {
        locale = getCookie(request, permLocale)
      }
      else {
        locale = getLocale(request)
      }

      console.log("locale:", locale)

      let response = new Response(locale, {
        status: 301
      })
      response.headers.append("Location", `https://sg.motochristo.com/${locale}`)
      response.headers.append("Set-Cookie", `${permLocale}=${locale}; path=/`);
      return response
    // }

    //Pass on requests for non-root paths to avoid redirect loops
    return await fetch(request)
  }

}

function getCookie(request, name) {
  let result = null
  let cookieString = request.headers.get('Cookie')
  if (cookieString) {
    let cookies = cookieString.split(';')
    cookies.forEach(cookie => {
      let cookieName = cookie.split('=')[0].trim()
      if (cookieName === name) {
        let cookieVal = cookie.split('=')[1]
        result = cookieVal
      }
    })
  }
  return result
}

function getLocale(request) {
  // If the region is not found in the lookup table then use the default otherwise return null
  // return countryMap[country][region] ? countryMap[country].default : null

  let locale;
  let country = request.cf.country;
  let region = country && request.cf.regionCode;

  console.log("country:", country);
  console.log("region:", region);

  if (countryMap[country]) {
    if (countryMap[country][region]) {
      locale = countryMap[country][region]
    }
    else {
      locale = countryMap[country].default
    }
  }
  console.log("locale:", locale)

  return locale ? locale : '';
}
```
