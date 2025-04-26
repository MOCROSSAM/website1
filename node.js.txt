
const express = require('express');
const stripe = require('stripe')('VOTRE_CLE_SECRETE_STRIPE');
const app = express();

app.use(express.json());
app.use(express.static('../ceinture-chauffante')); // Lien vers le dossier frontend

// Route pour Stripe
app.post('/creer-session-paiement', async (req, res) => {
    const { panier } = req.body;
    const line_items = panier.map(item => ({
        price_data: {
            currency: 'eur',
            product_data: { name: item.nom },
            unit_amount: item.prix * 100, // Stripe utilise des centimes
        },
        quantity: item.quantite,
    }));

    const session = await stripe.checkout.sessions.create({
        payment_method_types: ['card'],
        line_items,
        mode: 'payment',
        success_url: 'http://localhost:3000/confirmation.html',
        cancel_url: 'http://localhost:3000/panier.html',
    });

    res.json({ id: session.id });
});

app.listen(3000, () => console.log('Server running on port 3000'));