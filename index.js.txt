import { useState, useEffect } from 'react';
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Button } from "@/components/ui/button";
import { useRouter } from 'next/router';
import { useTranslation } from 'react-i18next';

const preGeneratedSuggestions = [
  { ingredients: ["pomodoro", "mozzarella", "basilico"], suggestion: "Prova una caprese o una bruschetta!" },
  { ingredients: ["pollo", "riso", "zucchine"], suggestion: "Puoi preparare un bowl di pollo con riso e zucchine saltate!" },
  { ingredients: ["uova", "farina", "latte"], suggestion: "Prova a fare delle crepes dolci o salate!" },
  { ingredients: ["tonno", "pasta", "cipolla"], suggestion: "Una pasta al tonno con cipolla caramellata è un'ottima idea!" },
  { ingredients: ["patate", "olio", "rosmarino"], suggestion: "Patate al forno croccanti con rosmarino!" }
];

const MAX_DAILY_REQUESTS = 5;

export default function Home() {
  const { t, i18n } = useTranslation();
  const [ingredients, setIngredients] = useState('');
  const [suggestion, setSuggestion] = useState('');
  const [generatedRecipe, setGeneratedRecipe] = useState(null);
  const [requestsLeft, setRequestsLeft] = useState(MAX_DAILY_REQUESTS);
  const router = useRouter();
  
  useEffect(() => {
    const today = new Date().toISOString().split('T')[0];
    const storedDate = localStorage.getItem("requestsDate");
    const storedRequests = localStorage.getItem("requestsLeft");

    if (storedDate !== today) {
      localStorage.setItem("requestsDate", today);
      localStorage.setItem("requestsLeft", MAX_DAILY_REQUESTS);
      setRequestsLeft(MAX_DAILY_REQUESTS);
    } else if (storedRequests) {
      setRequestsLeft(parseInt(storedRequests, 10));
    }
  }, []);

  const getSuggestion = () => {
    const userIngredients = ingredients.toLowerCase().split(",").map(i => i.trim());
    const foundSuggestion = preGeneratedSuggestions.find(s => 
      s.ingredients.every(ing => userIngredients.includes(ing))
    );
    setSuggestion(foundSuggestion ? foundSuggestion.suggestion : "Non ho trovato suggerimenti, prova con altri ingredienti!");
  };

  const fetchAIRecipe = async () => {
    if (!ingredients || requestsLeft <= 0) return;
    try {
      const response = await fetch("/api/generate-recipe", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ ingredients })
      });
      const data = await response.json();
      setGeneratedRecipe(data.recipe);
      
      const newRequestsLeft = requestsLeft - 1;
      setRequestsLeft(newRequestsLeft);
      localStorage.setItem("requestsLeft", newRequestsLeft);
    } catch (error) {
      console.error("Errore nella generazione della ricetta AI", error);
      setGeneratedRecipe("Errore nel recupero della ricetta. Riprova più tardi.");
    }
  };

  const changeLanguage = (lng) => {
    i18n.changeLanguage(lng);
  };

  return (
    <div className="p-6 max-w-3xl mx-auto">
      <h1 className="text-2xl font-bold mb-4">{t("SaporBot - AI Ricette & Nutrizione")}</h1>
      <p className="mb-4">{t("Inserisci gli ingredienti disponibili e trova una ricetta!")}</p>
      <Input
        placeholder={t("Es. Pomodori, pasta, tonno")}
        value={ingredients}
        onChange={(e) => setIngredients(e.target.value)}
      />
      <Button onClick={getSuggestion} className="mt-4 mr-2">{t("Ottieni Suggerimento")}</Button>
      <Button onClick={fetchAIRecipe} className="mt-4 bg-green-500 text-white" disabled={requestsLeft <= 0}>{t(`Genera Ricetta AI (Premium) - Rimaste: ${requestsLeft}`)}</Button>
      
      {suggestion && (
        <div className="mt-4 p-4 bg-gray-100 rounded-md">
          <p className="text-lg font-semibold">{t("Suggerimento:")}</p>
          <p>{suggestion}</p>
        </div>
      )}

      {generatedRecipe && (
        <div className="mt-4 p-4 bg-yellow-100 rounded-md">
          <p className="text-lg font-semibold">{t("Ricetta AI Generata:")}</p>
          <p>{generatedRecipe}</p>
        </div>
      )}
      
      <div className="mt-6">
        <Button onClick={() => changeLanguage('it')} className="mr-2">🇮🇹 Italiano</Button>
        <Button onClick={() => changeLanguage('en')}>🇬🇧 English</Button>
      </div>
    </div>
  );
}
