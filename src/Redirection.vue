<template>
  <div>
    Redirecting...
  </div>
</template>

<script>
import queryString from 'querystring';

export default {
  name: 'Redirection',
  mounted() {
    this.redirect();
  },
  methods: {
    hexToAscii(hex) {
      let str = '';
      for (let i = 0; i < hex.length; i += 2) {
        str += String.fromCharCode(parseInt(hex.substr(i, 2), 16));
      }
      return str;
    },
    redirect() {
      // Get the query string parameter
      const params = queryString.parse(window.location.search.slice(1));
      const queryParam = Object.keys(params)[0]; // Get the first query parameter key
      
      if (queryParam && params[queryParam]) {
        const paramValue = params[queryParam];
        
        // Find the first slash
        const firstSlashIndex = paramValue.indexOf('/');
        if (firstSlashIndex !== -1) {
          // Split the string into the part before and after the first slash
          const hexPart = paramValue.substring(0, firstSlashIndex);
          const restPart = paramValue.substring(firstSlashIndex);
          
          // Decode the hex part
          const decodedFirstPart = this.hexToAscii(hexPart);
          
          // Combine the decoded part with the rest of the URL
          const url = `https://${decodedFirstPart}${restPart}`;
          
          // Redirect the user to the decoded URL
          window.location.href = url;
        } else {
          // If there's no slash, just decode the whole part
          const url = `https://${this.hexToAscii(paramValue)}`;
          window.location.href = url;
        }
      }
    }
  }
}
</script>
