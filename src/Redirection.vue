<template>
  <div>
    <!-- This template can be empty or contain any UI elements you need -->
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      primaryHost: 'https://mailflow.v4mediasolutions.in',
      secondaryHost: 'https://emailtracking.v4mediasolutions.in',
      userIp: '',
      userAgent: navigator.userAgent,
      trackingParameter: '',
      type: '',
      tracking: '',
    };
  },
  async mounted() {
    try {
      await this.fetchRequestDetails();
      await this.handleType();
    } catch (error) {
      console.error('Error during mounted lifecycle:', error);
    }
  },
  methods: {
    async fetchRequestDetails() {
  try {
    console.log('fetchRequestDetails called');
    const ipResponse = await axios.get('https://api.ipify.org?format=json');
    console.log('IP Response:', ipResponse.data.ip);
    this.userIp = ipResponse.data.ip;

    const url = new URL(window.location.href);
    console.log('URL:', url);

    this.trackingParameter = null;

    if (url.searchParams.toString()) {
      console.log('URL has query params:', url.searchParams.toString());
      const firstQueryParam = url.searchParams.entries().next().value;
      if (firstQueryParam) {
        this.trackingParameter = firstQueryParam[1];
        console.log('Tracking Parameter from query:', this.trackingParameter);
      }
    } else {
      this.trackingParameter = url.pathname.split('/').pop();
      console.log('Tracking Parameter from path:', this.trackingParameter);
    }

    if (this.trackingParameter) {
      console.log('Calling extractTrackingInfo');
      this.extractTrackingInfo();
    }
  } catch (error) {
    console.error('Error fetching request details:', error);
  }
},
    extractTrackingInfo() {
      try {
        const parts = this.trackingParameter.split('__')[1]?.split('--')[1]?.split('_') || [];
        this.type = parts[1] ? this.decodeType(parts[1]) : 'unknown';
        this.tracking = parts[2] || 'unknown';
      } catch (error) {
        console.error('Error extracting tracking info:', error);
      }
    },
    decodeType(hexString) {
      try {
        let str = '';
        for (let i = 0; i < hexString.length; i += 2) {
          str += String.fromCharCode(parseInt(hexString.substr(i, 2), 16));
        }
        return str.split('|')[1] || 'unknown';
      } catch (error) {
        console.error('Error decoding type:', error);
        return 'unknown';
      }
    },
    async handleType() {
      if (this.type === 'opentrack') {
        await this.trackOpen();
      } else if (this.type === 'image') {
        await this.fetchImage();
      } else if (this.type === 'offer' || this.type === 'unsub') {
        await this.handleRedirection();
      }
    },
    async trackOpen() {
      try {
        const response = await axios.get(`${this.primaryHost}/tracking/open.php`, {
          params: { userip: this.userIp, useragent: this.userAgent, tracking: this.tracking }
        });

        if (response.data.includes('cURL Error')) {
          await axios.get(`${this.secondaryHost}/tracking/open.php`, {
            params: { userip: this.userIp, useragent: this.userAgent, tracking: this.tracking }
          });
        }
      } catch (error) {
        console.error('Failed to track email open:', error);
      }
    },
    async fetchImage() {
      try {
        const primaryUrl = `${this.primaryHost}/tracking/image.php?tracking=${encodeURIComponent(this.tracking)}`;
        const secondaryUrl = `${this.secondaryHost}/tracking/image.php?tracking=${encodeURIComponent(this.tracking)}`;
      
        let response = await fetch(primaryUrl);
      
        if (!response.ok) {
          response = await fetch(secondaryUrl);
        }
      
        if (response.ok) {
          const blob = await response.blob();
          const objectURL = URL.createObjectURL(blob);
          const img = document.createElement('img');
          img.src = objectURL;
          img.alt = 'Image'; // Provide a descriptive alt text
          document.body.appendChild(img);
        
          // Clean up object URL when image is no longer needed
          img.onload = () => URL.revokeObjectURL(objectURL);
        } else {
          // Handle error, e.g., display an error message
          console.error('Error fetching image:', response.statusText);
          document.body.textContent = 'Error: Unable to fetch image.';
        }
      } catch (error) {
        console.error('Error in fetchImage:', error);
        document.body.textContent = 'Error: Unable to fetch image.';
      }
    },
    async handleRedirection() {
      try {
        let redirectUrl = await axios.get(`${this.primaryHost}/tracking/redirect.php`, {
          params: { userip: this.userIp, useragent: this.userAgent, type: this.type, tracking: this.tracking }
        });

        if (redirectUrl.data.includes('cURL Error')) {
          redirectUrl = await axios.get(`${this.secondaryHost}/tracking/redirect.php`, {
            params: { userip: this.userIp, useragent: this.userAgent, type: this.type, tracking: this.tracking }
          });
        }

        if (redirectUrl.data) {
          window.location.href = redirectUrl.data.trim();
        } else {
          console.error('Failed to retrieve redirection URL.');
        }
      } catch (error) {
        console.error('Failed to handle redirection:', error);
      }
    }
  }
};
</script>

<style scoped>
/* Add any styles here */
</style>