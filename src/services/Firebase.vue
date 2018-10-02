<script>
const firebase = require("firebase/app");
// Required for side-effects
require("firebase/firestore");

firebase.initializeApp({
  apiKey: 'AIzaSyAQMZlW3bLkvc1PkRJ1H6b1Wz4J3gzw3pY',
  authDomain: 'epic-game-nights',
  projectId: 'epic-game-nights'
});

// Initialize Cloud Firestore through Firebase
var db = firebase.firestore();

// Disable deprecated features
db.settings({
  timestampsInSnapshots: true
});

export default {
  name: 'Firebase',
  db,
  const: firebase.firestore
}
</script>
