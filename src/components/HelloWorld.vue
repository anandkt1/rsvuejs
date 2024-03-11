<template>
  <!-- <h1>{{ msg }}</h1> -->
  <div class="container">
    <div style="text-align: center">
      <h1>Smart on FHIR - Patient Info</h1>
      <p><strong>Username:</strong> FHIR</p>
      <p><strong>Password:</strong> EpicFhir11!</p>
      <a
        class="btn btn-info"
        v-if="code == undefined"
        style="text-decoration: none"
        target="_blank"
        :href="authorizeLink"
      >
        Sign in
      </a>
      <hr />
    </div>
    <div v-if="accesstoken">
     
      <strong>: </strong>{{ patientdata}}<br />
    </div> 
     
     
      
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      code: "",
      accesstoken: "",
      patient: "",
      patientdata: {},
      clientId: "11c53e00-6d5c-4a40-8a3f-ccc196fa1382", // Replace with your client id
      redirect: import.meta.env.PROD
        ? "https://lucid-wozniak-940eae.netlify.app"
        : "http://localhost:3000",
    };
  },
  computed: {
    authorizeLink() {
      return `https://fhir.epic.com/interconnect-fhir-oauth/oauth2/authorize?response_type=code&redirect_uri=${this.redirect}&client_id=${this.clientId}&state=1234&scope=patient.read, appointment.find`;
    },
  },
  async mounted() {
    console.log(this.$route.query.code);
    this.code = this.$route.query.code;
    if (this.code != undefined) {
      const params = new URLSearchParams();
      params.append("grant_type", "authorization_code");
      params.append("redirect_uri", this.redirect);
      params.append("code", this.code);
      params.append("client_id", this.clientId);
      params.append("state", "1234"); 
      


      const config = {
        headers: {
          "Content-Type": "application/x-www-form-urlencoded",            
        },
      };
      await axios
        .post(
          "https://fhir.epic.com/interconnect-fhir-oauth/oauth2/token",
          params, 
          config 
        )
        .then((response) => {
          console.log(response.data);
          this.accesstoken = response.data.access_token;
          this.patient = response.data;
        })
        .catch(function (error) {
          console.log(error);
        });
    }

    if (this.accesstoken != "") {
      await axios
        .get(
          `https://fhir.epic.com/interconnect-fhir-oauth/api/FHIR/STU3/Appointment/$find/{"resourceType":"Parameters","parameter":[{"name":"patient","resource":{"resourceType":"Patient","extension":[{"valueCode":"M","url":"http://hl7.org/fhir/us/core/StructureDefinition/us-core-birthsex"},{"extension":[{"valueCoding":{"system":"http://hl7.org/fhir/us/core/ValueSet/omb-race-category","code":"2106-3","display":"White"},"url":"ombCategory"},{"valueString":"White","url":"text"}],"url":"http://hl7.org/fhir/us/core/StructureDefinition/us-core-race"},{"extension":[{"valueCoding":{"system":"http://hl7.org/fhir/us/core/ValueSet/omb-ethnicity-category","code":"UNK","display":"Unknown"},"url":"ombCategory"},{"valueString":"Unknown","url":"text"}],"url":"http://hl7.org/fhir/us/core/StructureDefinition/us-core-ethnicity"}],"identifier":[{"use":"usual","type":{"text":"EPIC"},"system":"urn:oid:1.2.840.114350.1.1","value":"E3423"},{"use":"usual","type":{"text":"MRN"},"system":"urn:oid:1.2.840.114350.1.13.0.1.7.5.737384.14","value":"203177"}],"active":"true","name":[{"use":"usual","text":"Correct Professional Billing","family":"Professional Billing","given":["Correct"]}],"telecom":[{"system":"phone","value":"608-271-9000","use":"home"},{"system":"phone","value":"608-271-9000","use":"work"}],"gender":"male","birthDate":"1983-06-08","deceasedBoolean":false,"address":[{"use":"home","line":["1979 Milky Way"],"city":"VERONA","district":"DANE","state":"WI","postalCode":"53593","country":"US"}],"maritalStatus":{"text":"Single"},"communication":[{"language":{"coding":[{"system":"http://hl7.org/fhir/ValueSet/languages","code":"en","display":"English"}],"text":"English"},"preferred":"true"}],"generalPractitioner":[{"reference":"https://apporchard.epic.com/interconnect-aocurprd-oauth/api/FHIR/STU3/Practitioner/eM5CWtq15N0WJeuCet5bJlQ3","display":"Physician Family Medicine, MD"}],"managingOrganization":{"reference":"https://apporchard.epic.com/interconnect-aocurprd-oauth/api/FHIR/STU3/Organization/enRyWnSP963FYDpoks4NHOA3","display":"Epic Hospital System"}}},{"name":"startTime","valueDateTime":"2024-05-01T13:00:00Z"},{"name":"endTime","valueDateTime":"2028-05-05T22:00:00Z"},{"name":"serviceType","valueCodeableConcept":{"coding":[{"system":"urn:oid:1.2.840.114350.1.13.0.1.7.3.808267.11","code":"95014","display":"Office Visit"}]}},{"name":"indications","valueCodeableConcept":{"coding":[{"system":"urn:oid:2.16.840.1.113883.6.96","code":"46866001","display":"Fracture of lower limb (disorder)"},{"system":"urn:oid:2.16.840.1.113883.6.90","code":"S82.90XA","display":"Broken leg"},{"system":"urn:oid:1.2.840.114350.1.13.861.1.7.2.696871","code":"121346631","display":"Broken leg"}],"text":"Broken leg"}},{"name":"location-reference","valueReference":{"reference":"https://apporchard.epic.com/interconnect-aocurprd-oauth/api/FHIR/STU3/Location/e4W4rmGe9QzuGm2Dy4NBqVc0KDe6yGld6HW95UuN-Qd03"}}]}`,
          { headers: {'Content-Type': 'application/json;charset=UTF-8', 
                      'Access-Control-Allow-Origin': '*', 
                      'Access-Control-Allow-Headers': '*', 
                      'Access-Control-Allow-Credentials': true,
                      'Sec-Fetch-Mode': no-cors,
                      'Sec-Fetch-Site': same-site, 
                       Authorization: `Bearer ${this.accesstoken}`  } }
        )
        .then((response) => {
          console.log(response.data);
          this.patientdata = response.data.patientdata;
         
        })
        .catch((error) => {
          console.log(error);
        });
    }
  },
};
//  const state = reactive({ count: 0 })
</script>

<style scoped>
/* a {
  color: #42b983;
} */
</style>
