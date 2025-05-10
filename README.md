// firebase.js
import { initializeApp } from "firebase/app";
import { getAuth } from "firebase/auth";
import { getFirestore } from "firebase/firestore";

const firebaseConfig = {
  apiKey: "AIzaSyBVbq8C3OaORt6yU1zWO8E7c3y7Y2VfQOo",
  authDomain: "massage-5800c.firebaseapp.com",
  projectId: "massage-5800c",
  storageBucket: "massage-5800c.firebasestorage.app",
  messagingSenderId: "796344060446",
  appId: "1:796344060446:web:b4514891fc2516513ffc0f",
  measurementId: "G-MVYC8GFX13"
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const firestore = getFirestore(app);


// App.jsx
import React from "react";
import { BrowserRouter as Router, Routes, Route } from "react-router-dom";
import ConfirmCodePage from "./ConfirmCodePage";
import Dashboard from "./Dashboard";

export default function App() {
  return (
    <Router>
      <Routes>
        <Route path="/" element={<ConfirmCodePage />} />
        <Route path="/dashboard" element={<Dashboard />} />
      </Routes>
    </Router>
  );
}


// ConfirmCodePage.jsx
import React, { useState } from "react";
import { useNavigate } from "react-router-dom";

export default function ConfirmCodePage() {
  const [code, setCode] = useState("");
  const navigate = useNavigate();

  const handleConfirm = () => {
    localStorage.setItem("confirmedCode", code);
    navigate("/dashboard");
  };

  return (
    <div style={{ textAlign: "center", marginTop: 100 }}>
      <h1>Your Code</h1>
      <input
        type="text"
        placeholder="Enter Your Code"
        value={code}
        onChange={(e) => setCode(e.target.value)}
        style={{ padding: 10, width: 250, marginBottom: 20 }}
      />
      <br />
      <button
        onClick={handleConfirm}
        style={{
          padding: 10,
          width: 150,
          backgroundColor: "blue",
          color: "white",
          border: "none",
          borderRadius: 5
        }}
      >
        Confirm
      </button>
    </div>
  );
}


// Dashboard.jsx
import React, { useState, useEffect } from "react";
import { auth, firestore } from "./firebase";
import { signOut } from "firebase/auth";
import { collection, getDocs, query, where } from "firebase/firestore";
import { useNavigate } from "react-router-dom";
import { motion } from "framer-motion";

export default function Dashboard() {
  const [darkMode, setDarkMode] = useState(false);
  const [searchUID, setSearchUID] = useState("");
  const [searchResult, setSearchResult] = useState(null);
  const [userUID, setUserUID] = useState("");
  const [userCode, setUserCode] = useState("");
  const navigate = useNavigate();

  useEffect(() => {
    const user = auth.currentUser;
    if (user) {
      setUserUID(user.uid);
      fetchUserCode(user.uid);
    }
    const confirmedCode = localStorage.getItem("confirmedCode");
    if (confirmedCode) {
      setUserCode(confirmedCode);
    }
  }, []);

  const fetchUserCode = async (uid) => {
    const q = query(collection(firestore, "users"), where("uid", "==", uid));
    const querySnapshot = await getDocs(q);
    querySnapshot.forEach((doc) => {
      setUserCode(doc.data().code);
    });
  };

  const handleLogout = () => {
    signOut(auth);
  };

  const handleSearch = async () => {
    const q = query(collection(firestore, "users"), where("uid", "==", searchUID));
    const querySnapshot = await getDocs(q);
    if (!querySnapshot.empty) {
      querySnapshot.forEach((doc) => {
        setSearchResult({ uid: doc.data().uid, name: doc.data().name });
      });
    } else {
      setSearchResult(null);
    }
  };

  const handleConnect = () => {
    navigate(`/chat?user1=${userUID}&user2=${searchResult.uid}`);
  };

  return (
    <div className={darkMode ? "dark" : "light"} style={{ padding: 20 }}>
      <header style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
        <h1>Contact List Dashboard</h1>
        <div>
          <button onClick={() => setDarkMode(!darkMode)}>
            {darkMode ? "Light Mode" : "Dark Mode"}
          </button>
          <button onClick={handleLogout}>Logout</button>
        </div>
      </header>

      <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} transition={{ duration: 0.5 }}>
        <p>Your Code: <strong>{userCode}</strong></p>
        <p>Your UID: <strong>{userUID}</strong></p>
      </motion.div>

      <motion.div initial={{ y: 50, opacity: 0 }} animate={{ y: 0, opacity: 1 }} transition={{ duration: 0.5 }}>
        <input
          type="text"
          placeholder="Search another UID"
          value={searchUID}
          onChange={(e) => setSearchUID(e.target.value)}
          style={{ padding: 10, marginRight: 10 }}
        />
        <button onClick={handleSearch}>Search</button>
      </motion.div>

      {searchResult && (
        <motion.div initial={{ scale: 0.8 }} animate={{ scale: 1 }} transition={{ duration: 0.3 }}>
          <p>Name: {searchResult.name}</p>
          <p>UID: {searchResult.uid}</p>
          <button
            onClick={handleConnect}
            style={{ backgroundColor: "blue", color: "white", padding: 10, border: "none", borderRadius: 5 }}
          >
            Connect
          </button>
        </motion.div>
      )}
    </div>
  );
}
