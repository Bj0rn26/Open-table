# Open-table
npx react-native init 
DinnerSchedulerApp
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​npm install @react-native-firebase/app 
@react-native-firebase/auth @react-native-firebase/firestore react-native-calendars react-native-contacts
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​{
  "uid": "unique_user_id",
  "name": "John Doe",
  "email": "john@example.com"
}
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​{
  "id": "dinner_id",
  "date": "2024-10-15",
  "host": "host_user_id",
  "seats": 5,
  "host_provisions": ["Pizza", "Salad"],
  "invitees": [
    { "userId": "friend_user_id", "status": "pending" }  // or "accepted", "declined"
  ],
  "guest_contributions": [
    { "userId": "friend_user_id", "contribution": "Drinks" }
  ]
}
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​

// Import the functions you need from the SDKs you need
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";
// TODO: Add SDKs for Firebase products that you want to use
// https://firebase.google.com/docs/web/setup#available-libraries

// Your web app's Firebase configuration
// For Firebase JS SDK v7.20.0 and later, measurementId is optional
const firebaseConfig = {
  apiKey: "AIzaSyBNucKbeN_JyB4sGkuvusorH1fEmdWT6ls",
  authDomain: "opentable-e4182.firebaseapp.com",
  projectId: "opentable-e4182",
  storageBucket: "opentable-e4182.firebasestorage.app",
  messagingSenderId: "272890362595",
  appId: "1:272890362595:web:cb24ea75e78e7dc54663d6",
  measurementId: "G-98GX9SEG1E"
};

// Initialize Firebase
const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​import auth from '@react-native-firebase/auth';

const login = (email, password) => {
  auth().signInWithEmailAndPassword(email, password)
    .then(() => console.log('User signed in!'))
    .catch(error => console.error(error));
};
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​import firestore from '@react-native-firebase/firestore';
const user = auth().currentUser;

// Hosted dinners
const hostedDinners = await firestore().collection('dinners')
  .where('host', '==', user.uid)
  .get();

// Invited dinners (pending or accepted)
const invitedDinners = await firestore().collection('dinners')
  .where('invitees', 'array-contains', { userId: user.uid, status: 'pending' })
  .get();
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​import { Calendar } from 'react-native-calendars';

<Calendar onDayPress={(day) => setSelectedDate(day.dateString)} />
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​firestore().collection('dinners').add({
  date: selectedDate,
  host: user.uid,
  seats: seats,
  host_provisions: hostProvisions, // e.g., ["Pizza", "Salad"]
  invitees: selectedInvitees.map(invitee => ({ userId: invitee.uid, status: 'pending' })),
  guest_contributions: [],
});
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​firestore().collection('dinners').doc(dinnerId).update({
  invitees: firestore.FieldValue.arrayRemove({ userId: user.uid, status: 'pending' }),
}).then(() => {
  firestore().collection('dinners').doc(dinnerId).update({
    invitees: firestore.FieldValue.arrayUnion({ userId: user.uid, status: 'accepted' }),
  });
});
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​firestore().collection('dinners').doc(dinnerId).update({
  guest_contributions: firestore.FieldValue.arrayUnion({
    userId: user.uid,
    contribution: 'Dessert',
  }),
});
​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​​