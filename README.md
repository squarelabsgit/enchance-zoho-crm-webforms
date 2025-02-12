# How to Enhance Zoho CRM Webforms
Blog Post: https://www.squarelabs.com.au/post/how-to-enhance-zoho-crm-webforms

YouTube: https://youtu.be/VcPJBISEOPU

Optimising web forms is crucial for improving user experience and increasing conversion rates. While Zoho CRM Webforms do not currently offer built-in functionality for pre-filling fields dynamically, we can enhance Zoho crm webforms by modifying the supplied code.

This article will explore three powerful techniques to achieve this by using URL parameters, adding IP address geolocation and Browser/Device geolocation. We'll dive into the code and explain how each method works, along with their pros and cons, providing you with custom solutions to implement in your Zoho CRM Webforms.

## Getting the webform code
1. In Zoho CRM head to Settings > Channels > Webforms
2. Click on elipsis (...) next to the webform you want to enhance
3. Click Publish Options
4. Ensure Publishing Format is source code and click copy

You can open this code inside a text editor to modify it.

## 1. Pre filling Webforms with URL Parameters
URL parameters are a simple yet effective way to pass information to your web form. This method is particularly useful when you want to pre-populate fields based on data from external sources or previous user interactions.

It starts with locating the field HTML element IDs within the webform. Most of the Zoho CRM system created fields have the ID that matches the API Name however if you have custom fields in the webform they will have a different ID.

In this below example snippet from the Zoho CRM Webform we are wanting to locate the id attribute within the HTML for the fields we want to pre-fill.
1. Country (System field) has the id of "Country"
2. Product (Custom field) has the id of "LEADCF8"

```html
<div class='zcwf_row wfrm_fld_dpNn'>
  <div class='zcwf_col_lab'>
    <label for='Country'>Country</label>
  </div>
  <div class='zcwf_col_fld'>
     <input type='text' id='Country' aria-required='false' aria-label='Country' name='Country' aria-valuemax='100' maxlength='100' value='United&#x20;States'></input>
     <div class='zcwf_col_help'></div>
  </div>
</div>
<div class='zcwf_row'>
  <div class='zcwf_col_lab'>
    <label for='LEADCF8'>Product</label>
  </div>
  <div class='zcwf_col_fld'>
     <input type='text' id='LEADCF8' aria-required='false' aria-label='LEADCF8' name='LEADCF8' aria-valuemax='255' maxlength='255'></input>
     <div class='zcwf_col_help'></div>
  </div>
</div>
```

Adding to your Zoho CRM provided webform code.

At the bottom of your webform code just before the closing script tag </script> we want to add in the following code snippet that incorporates two functions. One that will perform the pre-filling action and another that will trigger when the window is loaded to call the function to pre-fill the fields.

```js
function prefillUrlParams() {
  // Map URL parameters to their corresponding form field IDs
	const paramToFieldMap = {
		source: "Lead_Source",
		product: "LEADCF9",
		};
	// Get URL parameters
  const urlParams = new URLSearchParams(window.location.search);
  // Loop through the map to process each URL parameter
 for (const [urlParam, fieldId] of Object.entries(paramToFieldMap)) {
    console.log(`Processing parameter: ${urlParam}`);
    if (urlParams.has(urlParam)) {
      const value = urlParams.get(urlParam);
      if (value === null || value.trim() === "") {
        console.warn(`Parameter "${urlParam}" exists in the URL but has no value.`);
        continue;
      }
      const element = document.getElementById(fieldId);
      if (element) {
        element.value = value; 
        console.log(`Prefilled "${fieldId}" with value from "${urlParam}": ${value}`);
      } else {
        console.warn(`No element found with ID: "${fieldId}"`);
      }
    } else {
      console.info(`Parameter "${urlParam}" not found in the URL.`);
    }
  }
}

window.onload = function () {
  prefillUrlParams();
};
```

Inside the code you can update the paramToFieldMap object with what the parameter is and the ID of the field you are searching for in the form. e.g. In the example we have "source" and "Lead_Source".

https://examplewebsite.com/contact-us?source=Website&product=ProductCode

In the example URL above the Lead Source field will be pre-filled with "Website" and the Product field will be pre-filled with "Product Code".

Pros:
1. Easy to implement and customise
2. Works well with marketing campaigns and external links
3. No additional API calls required

Cons:
1. Limited to information that can be passed through the URL
2. URL parameters are visible to the user, which may not be suitable for sensitive information

## 2. Pre-filling Country based on IP Address
Now that we know how to pre-fill the field values we can expand this to learning more about the user submitting the form. One that always comes to mind is location, where is the user submitting the form located.

By using a free location api service Free IP API service (https://freeipapi.com/) we can get the country or other location data such as State and Postcode based on their IP address.

Using the below code snippet and installing the same way as the above pre-filling function we can pre-fill the country field.

```js
function getCountryFromIP() {
  fetch("https://freeipapi.com/api/json/")
    .then((response) => response.json())
    .then((data) => {
      const country = data.countryName;
      const countryElement = document.getElementById("Country");
      if (countryElement) {
        countryElement.value = country;
        console.log(`Country: ${country} set based on IP location`);
      } else {
        console.warn("No element found with ID: Country");
      }
    });
}

window.onload = function () {
    getCountryFromIP();
};
```

Pros:
1. No user interaction required
2. Generally accurate at the country level
3. No browser permissions needed

Cons:
1. Less accurate than Browser geolocation API
2. May not be accurate if the user is using a VPN

## 3. Pre-filling Country based on browser geolocation
For more accurate location data, you can use the browser's Geolocation API to get the user's browser geolocation and then reverse geocode them to determine the country. The browser geolocation uses an array of sources such as IP, Cell Tower Triangulation, Wifi-Positioning and Bluetooth Beacons/Signals to provide a highly accurate location especial if the user is using a mobile device!

We can then use the free Open Street Map reverse geocode API to get address based on the Latitude and Longitude returned.

When using the Geolocation API in a browser it will popup and ask the user if they give permission for your website to access your location.

```js
function setCountryBasedOnGPSLocation() {
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
    (position) => {
      const { latitude, longitude } = position.coords;
      fetch(`https://nominatim.openstreetmap.org/reverse?lat=${latitude}&lon=${longitude}&format=jsonv2`)
        .then((response) => response.json())
        .then((data) => {
        const address = data.address;
        const country = address.country;
            const countryElement = document.getElementById("Country");
            if (countryElement) {
              countryElement.value = country;
              console.log(`Country: ${country} updated based on geolocation location`);
            } else {
              console.warn("No element found with ID: Country");
            }
          })
          .catch((error) => console.error("Error fetching country data based on GPS:", error));
      },
      (error) => {
        console.error("Geolocation error:", error.message);
      }
    );
  } else {
    console.error("Geolocation is not supported by this browser.");
  }
}

window.onload = function () {
  setCountryBasedOnLocation();
};
```

Pros:
1. More accurate location data
2. Can provide more detailed location information if needed such as street, city & state.

Cons:
1. Requires user permission, which may be declined
2. May take longer to retrieve the location

## Bonus: Combining Location Methods
For the best user experience, you can combine these methods. Start with the IP-based location and then attempt to refine it with the browser geolocation data if the user allows it. 

Here's an updated function that implements this approach.

```js
function  getCountryFromIP()  {
  fetch("https://freeipapi.com/api/json/")
    .then((response)  =>  response.json())
    .then((data)  =>  {
      const  country  =  data.countryName;
      const  countryElement  =  document.getElementById("Country");
      if  (countryElement)  {
        countryElement.value  =  country;
        console.log(`Country:  ${country}  set  based  on  IP  location`);
      }  else  {
    console.warn("No  element  found  with  ID:  Country");
  }
    });
}

function setCountryBasedOnLocation() {
  // Step 1: Prefill using Free IP API based on the IP address
  getCountryFromIP();
  // Step 2: If the user accepts, update using GPS-based location
  if (navigator.geolocation) {
    navigator.geolocation.getCurrentPosition(
      (position) => {
        const { latitude, longitude } = position.coords;
        fetch(`https://nominatim.openstreetmap.org/reverse?lat=${latitude}&lon=${longitude}&format=jsonv2`)
          .then((response) => response.json())
          .then((data) => {
            const address = data.address;
            const country = address.country;
            const countryElement = document.getElementById("Country");
            if (countryElement) {
              countryElement.value = country;
              console.log(`Country: ${country} updated based on geolocation location`);
            } else {
              console.warn("No element found with ID: Country");
            }
          })
          .catch((error) => console.error("Error fetching country data based on GPS:", error));
      },
      (error) => {
        console.error("Geolocation error:", error.message);
      }
    );
  } else {
    console.error("Geolocation is not supported by this browser.");
  }
}

window.onload = function () {
  setCountryBasedOnLocation();
};
```

This combined approach ensures that you always have some location data (from the IP address) while still providing the option for more accurate geolocation based location if the user allows it.

## Important Note
If you are combining the Pre-fill via URL Parameters and Country location functions into your CRM Webform code you only need to include a single function to trigger both of these functions on loading of the page by adding the function name like below. Also ensure that this function is at the bottom of the webform code right before the closing script tag </script>.

```js
window.onload = function () {
	prefillUrlParams();
  setCountryBasedOnLocation();
};
```

## Summary
By implementing these techniques into your webforms you can create smarter, more user-friendly forms that reduce friction and boost conversions as fields that are pre-filled can be hidden from the user.

Whether you find these implementations too technical or you're looking for a bespoke form enhancement tailored to your specific requirements, our team of experts is ready to assist. Contact us and we can help you implement these solutions or create custom enhancements that perfectly align with your business processes and user experience goals.

Need Help? [Contact us!](https://www.squarelabs.com.au/contact-us)

<a href="http://www.youtube.com/watch?feature=player_embedded&v=VcPJBISEOPU" target="_blank"><img src="http://img.youtube.com/vi/VcPJBISEOPU/0.jpg" 
alt="YouTube Video" width="240" height="180" border="10" /></a>

## Resources
https://help.zoho.com/portal/en/kb/crm/connect-with-customers/webforms/articles/set-up-web-forms#Generate_Webforms

https://www.w3schools.com/html/tryit.asp?filename=tryhtml_editor

https://freeipapi.com/

https://nominatim.org/release-docs/latest/api/Reverse/




