# Génération de la bibliothèque et paquets
import tkinter as tk
from tkinter import ttk
import matplotlib.pyplot as plt
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import random
import folium
import webbrowser
import os

# Données 
vehicules = {
    101: {'position': [33.589886, -7.603869], 'vitesse': 45},  # Casablanca Centre
    102: {'position': [33.573110, -7.589843], 'vitesse': 60},  # Quartier des Hôpitaux
    103: {'position': [33.580000, -7.630000], 'vitesse': 30},  # Hay Hassani
}

# Simulation 
def simuler_latence(nb_vehicules, seuil_latence, duree):
    latences_moyennes = []
    alertes = []
    congestions = []
    for t in range(duree):
        latences = [random.uniform(10, 150) for _ in range(nb_vehicules)]
        moyenne = sum(latences) / nb_vehicules                     # Génération de la latence moyenne pour les trois véhicules
        nb_alertes = sum(1 for l in latences if l > seuil_latence) # Condition de déclenchement des alertes
        congestion = sum(1 for l in latences if l > 120)           # Condition de la congestion
        latences_moyennes.append(moyenne)
        alertes.append(nb_alertes)
        congestions.append(congestion)
    return latences_moyennes, alertes, congestions

# Affichage graphique 
def generer_graphique():
    try:
        nb_veh = int(entry_nb_veh.get())
        seuil = float(entry_seuil.get())
        duree = int(entry_duree.get())
    except ValueError:
        return

    latences, alertes, congestions = simuler_latence(nb_veh, seuil, duree)

    fig, axs = plt.subplots(3, 1, figsize=(7, 7))
    
    # Courbe de latence moyenne
    axs[0].plot(latences, label='Latence moyenne (ms)', color='blue')
    axs[0].axhline(y=seuil, color='red', linestyle='--', label=f'Seuil ({seuil} ms)')
    axs[0].set_ylabel('Latence (ms)')
    axs[0].set_title("Évolution de la latence moyenne")
    axs[0].grid(True)
    axs[0].legend()

    # Courbe des alertes de sécurité
    axs[1].plot(alertes, label='Alertes de sécurité', color='red')
    axs[1].set_ylabel('Alertes')
    axs[1].set_title("Nombre d'alertes de sécurité")
    axs[1].grid(True)

    # Courbe de congestion du réseau
    axs[2].plot(congestions, label='Congestion du réseau', color='green')
    axs[2].set_ylabel('Congestion')
    axs[2].set_xlabel('Temps (s)')
    axs[2].set_title("Congestion du réseau")
    axs[2].grid(True)

    plt.tight_layout()

    for widget in frame_plot.winfo_children():
        widget.destroy()

    canvas = FigureCanvasTkAgg(fig, master=frame_plot)
    canvas.draw()
    canvas.get_tk_widget().pack()

# GPS_Casablanca 
def afficher_carte():
    m = folium.Map(location=[33.589886, -7.603869], zoom_start=12)
    for vid, info in vehicules.items():
        folium.Marker(
            location=info['position'],
            popup=f"ID: {vid} | Vitesse: {info['vitesse']} km/h",
            icon=folium.Icon(color='blue', icon='car', prefix='fa')
        ).add_to(m)

    chemin_html = "carte_vehicules.html"
    m.save(chemin_html)
    webbrowser.open('file://' + os.path.realpath(chemin_html))

# Arrêt de la simulation 
def arreter_simulation():
    root.quit()

# Interface Graphique
root = tk.Tk()
root.title("Simulation d'un scénario de circulation réaliste à Casablanca")

frame_haut = ttk.Frame(root)
frame_haut.pack(padx=10, pady=10)

# Titre
label_titre = ttk.Label(
    frame_haut,
    text="Simulation d'un scénario de circulation réaliste à Casablanca",
    font=("Arial", 16, "bold")
)
label_titre.grid(row=0, column=0, columnspan=4, pady=10)

# Entrées dans l'interface graphique
entry_labels = ["Nombre de véhicules :", "Seuil de latence (ms) :", "Durée de simulation (s) :"]
entries = []
for i, label_text in enumerate(entry_labels):
    ttk.Label(frame_haut, text=label_text).grid(row=i+1, column=0, sticky="w")
    entry = ttk.Entry(frame_haut)
    entry.grid(row=i+1, column=1)
    entries.append(entry)

entry_nb_veh, entry_seuil, entry_duree = entries
# Fixer le nombre de véhicules à 3 et rendre le champ non modifiable
entry_nb_veh.insert(0, "3")
entry_nb_veh.config(state='readonly')
entry_seuil.insert(0, "100")
entry_duree.insert(0, "20")

# Boutons
ttk.Button(frame_haut, text="Lancer la simulation", command=generer_graphique).grid(row=1, column=2, padx=10)
ttk.Button(frame_haut, text="Afficher la carte", command=afficher_carte).grid(row=2, column=2, padx=10)
ttk.Button(frame_haut, text="Arrêter la simulation", command=arreter_simulation).grid(row=3, column=2, padx=10, pady=5)

# Graphiques
frame_plot = ttk.Frame(root)
frame_plot.pack(padx=10, pady=10)

root.mainloop()
