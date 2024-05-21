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
        if (params.q) {
          // Decode the hex-encoded query string
          const decodedQuery = this.hexToAscii(params.q);
          const url = `https://${decodedQuery}`;
          // Redirect the user to the decoded URL
          window.location.href = url;
        }
      }
    }
  }
  </script>
  