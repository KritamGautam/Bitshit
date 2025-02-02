# Bitshit
npm install gh-pages --save-dev
"homepage": "https://YOUR_USERNAME.github.io/YOUR_REPO_NAME",
   "scripts": {
     "predeploy": "npm run build",
     "deploy": "gh-pages -d build"
   }
   npm install express mongoose bcryptjs jsonwebtoken cors body-parser axios
   const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');
const cors = require('cors');
const jwt = require('jsonwebtoken');
const bcrypt = require('bcryptjs');
const axios = require('axios');

const app = express();
const PORT = 5000;

app.use(cors());
app.use(bodyParser.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/cryptoApp', { useNewUrlParser: true, useUnifiedTopology: true });

// User Schema
const UserSchema = new mongoose.Schema({
    username: String,
    password: String,
    wallet: { type: Map, of: Number, default: {} } // Wallet to store crypto balances
});
const User = mongoose.model('User', UserSchema);

// Middleware to verify JWT
function verifyToken(req, res, next) {
    const token = req.headers['authorization'];
    if (!token) return res.status(403).send('A token is required for authentication');
    try {
        req.user = jwt.verify(token.split(' ')[1], 'YOUR_SECRET_KEY');
        next();
    } catch (err) {
        return res.status(401).send('Invalid Token');
    }
}

// Routes

// Register User
app.post('/register', async (req, res) => {
    try {
        const hashedPassword = bcrypt.hashSync(req.body.password, 8);
        const user = new User({ username: req.body.username, password: hashedPassword });
        await user.save();
        res.status(201).send('User registered');
    } catch (error) {
        res.status(500).send(error);
    }
});

// Login User
app.post('/login', async (req, res) => {
    const user = await User.findOne({ username: req.body.username });
    if (user && bcrypt.compareSync(req.body.password, user.password)) {
        const token = jwt.sign({ userId: user._id }, 'YOUR_SECRET_KEY', { expiresIn: '24h' });
        res.json({ token });
    } else {
        res.status(401).send('Invalid credentials');
    }
});

// Get Crypto Prices (Using a public API like CoinGecko)
app.get('/crypto-prices', async (req, res) => {
    try {
        const response = await axios.get('https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum,dogecoin&vs_currencies=usd');
        res.json(response.data);
    } catch (error) {
        res.status(500).send('Error fetching crypto prices');
    }
});

// Buy Crypto
app.post('/buy-crypto', verifyToken, async (req, res) => {
    const { userId } = req.user;
    const { coin, amount } = req.body;

    try {
        const user = await User.findById(userId);
        if (!user) return res.status(404).send('User not found');

        // Update wallet balance
        user.wallet.set(coin, (user.wallet.get(coin) || 0) + amount);
        await user.save();

        res.json({ message: `Purchased ${amount} ${coin}`, wallet: user.wallet });
    } catch (error) {
        res.status(500).send(error);
    }
});

// Get User Wallet
app.get('/wallet', verifyToken, async (req, res) => {
    const { userId } = req.user;

    try {
        const user = await User.findById(userId);
        if (!user) return res.status(404).send('User not found');

        res.json({ wallet: user.wallet });
    } catch (error) {
        res.status(500).send(error);
    }
});

app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
npx create-react-app client
cd client
npm install axios react-router-dom
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import { BrowserRouter as Router, Route, Link, Routes } from 'react-router-dom';

function App() {
    const [cryptoPrices, setCryptoPrices] = useState({});
    const [wallet, setWallet] = useState({});
    const [token, setToken] = useState(localStorage.getItem('token') || '');

    useEffect(() => {
        if (token) {
            fetchCryptoPrices();
            fetchWallet();
        }
    }, [token]);

    const fetchCryptoPrices = async () => {
        try {
            const response = await axios.get('http://localhost:5000/crypto-prices');
            setCryptoPrices(response.data);
        } catch (error) {
            console.error('Error fetching crypto prices:', error);
        }
    };

    const fetchWallet = async () => {
        try {
            const response = await axios.get('http://localhost:5000/wallet', {
                headers: { Authorization: `Bearer ${token}` }
            });
            setWallet(response.data.wallet);
        } catch (error) {
            console.error('Error fetching wallet:', error);
        }
    };

    const buyCrypto = async (coin, amount) => {
        try {
            await axios.post(
                'http://localhost:5000/buy-crypto',
                { coin, amount },
                { headers: { Authorization: `Bearer ${token}` } }
            );
            fetchWallet(); // Refresh wallet after purchase
        } catch (error) {
            console.error('Error buying crypto:', error);
        }
    };

    return (
        <Router>
            <div>
                <nav>
                    <Link to="/">Home</Link>
                </nav>
                <Routes>
                    <Route
                        path="/"
                        element={
                            <div>
                                <h1>Crypto Prices</h1>
                                {Object.keys(cryptoPrices).map((coin) => (
                                    <div key={coin}>
                                        <h3>{coin.toUpperCase()}</h3>
                                        <p>${cryptoPrices[coin].usd}</p>
                                        <button onClick={() => buyCrypto(coin, 1)}>Buy 1 {coin.toUpperCase()}</button>
                                    </div>
                                ))}
                                <h2>Your Wallet</h2>
                                {Object.keys(wallet).map((coin) => (
                                    <div key={coin}>
                                        <p>{coin.toUpperCase()}: {wallet[coin]}</p>
                                    </div>
                                ))}
                            </div>
                        }
                    />
                </Routes>
            </div>
        </Router>
    );
}

export default App;
node server.js
cd client
   npm start
