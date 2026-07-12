"""
Simulation & détection de surcharge des bus (ETUS) à Oran - Juillet
"""
import streamlit as st
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

np.random.seed(42)

ANNEE = 2026
MOIS = 7
NB_JOURS = 31

LIGNES = {
    "L1 - Es Sénia <-> Centre Ville": 90,
    "L2 - Bir El Djir <-> Centre Ville": 90,
    "L3 - USTO <-> Front de Mer": 80,
    "L4 - El Hamri <-> Gambetta": 70,
    "L5 - Plage Les Andalouses (estivale)": 90,
    "L6 - Canastel <-> Centre Ville": 80,
}

SEUIL_SURCHARGE = 1.0
SEUIL_SURCHARGE_CRITIQUE = 1.3
DEPARTS_PAR_JOUR = 14
HEURE_DEBUT_SERVICE = 6
HEURE_FIN_SERVICE = 20


def facteur_horaire(heure):
    if heure in (7, 8, 9):
        return 1.3
    if heure in (17, 18, 19):
        return 1.4
    if heure in (12, 13, 14, 15):
        return 1.15
    return 0.75


def facteur_ligne_estivale(nom_ligne, heure):
    if "Plage" in nom_ligne or "Andalouses" in nom_ligne:
        return 1.6 if 11 <= heure <= 18 else 0.5
    return 1.0


def facteur_weekend(date):
    return 1.15 if date.weekday() in (4, 5) else 1.0


def generer_donnees_simulees():
    lignes_data = []
    for jour in range(1, NB_JOURS + 1):
        date = datetime(ANNEE, MOIS, jour)
        for ligne, capacite in LIGNES.items():
            heures_service = np.linspace(HEURE_DEBUT_SERVICE, HEURE_FIN_SERVICE, DEPARTS_PAR_JOUR)
            for h in heures_service:
                heure = int(round(h))
                base = capacite * np.random.uniform(0.5, 0.85)
                f_h = facteur_horaire(heure)
                f_l = facteur_ligne_estivale(ligne, heure)
                f_w = facteur_weekend(date)
                bruit = np.random.normal(1.0, 0.12)
                passagers = max(0, int(round(base * f_h * f_l * f_w * bruit)))
                lignes_data.append({
                    "ligne": ligne,
                    "date": date.strftime("%Y-%m-%d"),
                    "heure_depart": f"{heure:02d}:00",
                    "capacite_nominale": capacite,
                    "passagers_estimes": passagers,
                })
    return pd.DataFrame(lignes_data)


def detecter_surcharges(df):
    df = df.copy()
    df["taux_occupation"] = df["passagers_estimes"] / df["capacite_nominale"]

    def statut(t):
        if t >= SEUIL_SURCHARGE_CRITIQUE:
            return "SURCHARGE CRITIQUE"
        elif t >= SEUIL_SURCHARGE:
            return "SURCHARGE"
        elif t >= 0.8:
            return "CHARGE ELEVEE"
        return "NORMAL"

    df["statut"] = df["taux_occupation"].apply(statut)
    return df


def main():
    df = generer_donnees_simulees()
    df = detecter_surcharges(df)
    surcharges = df[df["statut"].isin(["SURCHARGE", "SURCHARGE CRITIQUE"])]

    print(f"Total trajets : {len(df)}")
    print(f"Trajets en surcharge : {len(surcharges)} ({100*len(surcharges)/len(df):.1f}%)")
    print(df.groupby("ligne")["taux_occupation"].mean().sort_values(ascending=False))

    df.to_csv("trajets_juillet_oran.csv", index=False)
    surcharges.to_csv("surcharges_detectees.csv", index=False)


if __name__ == "__main__":
    main()
