#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdbool.h>

#define MAX_TITRE 100
#define MAX_AUTEUR 100
//information de livres
typedef struct {
    char titre[MAX_TITRE];
    char auteur[MAX_AUTEUR];
    int isbn;
    bool disponible;
} Livre;
//listes des livres
typedef struct Noeud {
    Livre livre;
    struct Noeud* suivant;
} Noeud;
//tete de listes des livres
typedef struct {
    Noeud* tete;
} ListeLivres;
//files des livres
typedef struct {
    Noeud* premier;
    Noeud* dernier;
} FileLivres;

typedef struct NoeudArbre {
    Livre livre;
    struct NoeudArbre* gauche;
    struct NoeudArbre* droite;
} NoeudArbre;

typedef struct {
    NoeudArbre* racine;
} ArbreLivres;

void ajouterLivre(ListeLivres* listeLivres, Livre livre) {
    Noeud* nouveauNoeud = (Noeud*)malloc(sizeof(Noeud));
    nouveauNoeud->livre = livre;
    nouveauNoeud->suivant = NULL;

    if (listeLivres->tete == NULL) {
        listeLivres->tete = nouveauNoeud;
    } else {
        Noeud* noeudActuel = listeLivres->tete;
        while (noeudActuel->suivant != NULL) {
            noeudActuel = noeudActuel->suivant;
        }
        noeudActuel->suivant = nouveauNoeud;
    }
}


void supprimerLivre(ListeLivres* listeLivres, int isbn) {
    if (listeLivres->tete == NULL) {
        return;
    }

    if (listeLivres->tete->livre.isbn == isbn) {
        Noeud* noeudASupprimer = listeLivres->tete;
        listeLivres->tete = listeLivres->tete->suivant;
        free(noeudASupprimer);
        return;
    }

    Noeud* noeudActuel = listeLivres->tete;
    while (noeudActuel->suivant != NULL && noeudActuel->suivant->livre.isbn != isbn) {
        noeudActuel = noeudActuel->suivant;
    }

    if (noeudActuel->suivant != NULL) {
        Noeud* noeudASupprimer = noeudActuel->suivant;
        noeudActuel->suivant = noeudASupprimer->suivant;
        free(noeudASupprimer);
    }
}

void emprunterLivre(ListeLivres* listeLivres, FileLivres* fileAttente, int isbn) {
    Noeud* noeudActuel = listeLivres->tete;
    while (noeudActuel != NULL) {
        if (noeudActuel->livre.isbn == isbn) {
            if (noeudActuel->livre.disponible) {
                noeudActuel->livre.disponible = false;
                printf("\t\t\t\t\t\tLe livre '%s' est emprunte.\n", noeudActuel->livre.titre);
            } else {
                Noeud* nouveauNoeud = (Noeud*)malloc(sizeof(Noeud));
                nouveauNoeud->livre = noeudActuel->livre;
                nouveauNoeud->suivant = NULL;

                if (fileAttente->premier == NULL) {
                    fileAttente->premier = nouveauNoeud;
                    fileAttente->dernier = nouveauNoeud;
                } else {
                    fileAttente->dernier->suivant = nouveauNoeud;
                    fileAttente->dernier = nouveauNoeud;
                }
                printf("\t\t\t\t\t\tLe livre '%s' n est pas disponible. Vous avez est mis en file d attente.\n", noeudActuel->livre.titre);
            }
            return;
        }
        noeudActuel = noeudActuel->suivant;
    }
    printf("\t\t\t\t\t\tLe livre avec l ISBN %d n a pas trouve.\n", isbn);
}

void retournerLivre(ListeLivres* listeLivres, FileLivres* fileAttente, int isbn) {
    Noeud* noeudActuel = listeLivres->tete;
    while (noeudActuel != NULL) {
        if (noeudActuel->livre.isbn == isbn) {
            noeudActuel->livre.disponible = true;
            printf("\t\t\t\t\t\tLe livre '%s' a est retourne.\n", noeudActuel->livre.titre);

            if (fileAttente->premier != NULL) {
                Noeud* noeudEmprunteur = fileAttente->premier;
                noeudActuel->livre.disponible = false;
                printf("\t\t\t\t\t\tLe livre '%s' est emprunte par le prochain emprunteur en file d attente.\n", noeudActuel->livre.titre);
                fileAttente->premier = fileAttente->premier->suivant;
                free(noeudEmprunteur);
            }
            return;
        }
        noeudActuel = noeudActuel->suivant;
    }
    printf("\t\t\t\t\t\tLe livre avec ISBN %d n a pas trouve.\n", isbn);
}

void afficherLivresDepuisFichier(const char* nomFichier) {
    FILE* fichier = fopen(nomFichier, "r");
    if (fichier == NULL) {

        return;
    }

    char ligne[256];
    while (fgets(ligne, sizeof(ligne), fichier) != NULL) {
        Livre livre;
        sscanf(ligne, "%[^;];%[^;];%d;%d", livre.titre, livre.auteur, &livre.isbn, (int*)&livre.disponible);
        printf("\t\t\t\t\t\tTitre: %s, Auteur: %s, ISBN: %d, Disponible: %s\n", livre.titre, livre.auteur, livre.isbn, livre.disponible ? "Oui" : "Non");
    }

    fclose(fichier);
}

void sauvegarderLivres(ListeLivres* listeLivres) {
    FILE* fichier = fopen("livres.txt", "w");
    if (fichier == NULL) {

        return;
    }

    Noeud* noeudActuel = listeLivres->tete;
    while (noeudActuel != NULL) {
        fprintf(fichier, "%s;%s;%d;%d\n", noeudActuel->livre.titre, noeudActuel->livre.auteur, noeudActuel->livre.isbn, noeudActuel->livre.disponible);
        noeudActuel = noeudActuel->suivant;
    }

    fclose(fichier);

}

void chargerLivres(ListeLivres* listeLivres) {
    FILE* fichier = fopen("livres.txt", "r");
    if (fichier == NULL) {

        return;
    }

    char ligne[256];
    while (fgets(ligne, sizeof(ligne), fichier) != NULL) {
        Livre livre;
        sscanf(ligne, "%[^;];%[^;];%d;%d", livre.titre, livre.auteur, &livre.isbn, &livre.disponible);
        ajouterLivre(listeLivres, livre);
    }

    fclose(fichier);
}

void viderListe(ListeLivres* listeLivres) {
    Noeud* noeudActuel = listeLivres->tete;
    while (noeudActuel != NULL) {
        Noeud* noeudASupprimer = noeudActuel;
        noeudActuel = noeudActuel->suivant;
        free(noeudASupprimer);
    }
    listeLivres->tete = NULL;
}


int main() {
    ListeLivres listeLivres = {NULL};
    FileLivres fileAttente = {NULL, NULL};
    ArbreLivres arbreLivres = {NULL};

    int choix;
    do {
        printf("\n \t\t\t======================= Menu:============================\n");
        printf(" \t\t\t\t\t\t1. Ajouter un livre\n");
        printf(" \t\t\t\t\t\t2. Supprimer un livre\n");
        printf(" \t\t\t\t\t\t3. Emprunter un livre\n");
        printf(" \t\t\t\t\t\t4. Retourner un livre\n");
        printf(" \t\t\t\t\t\t5. Afficher les livres\n");
        printf(" \t\t\t\t\t\t0. Quitter\n");
        printf(" \t\t\t==========================================================\n");
        printf(" \t\t\t\t\t\tTaper Votre choix: ");
        scanf("%d", &choix);

        switch (choix) {
            case 1: {
                Livre livre;
                viderListe(&listeLivres);
                chargerLivres(&listeLivres);
                printf("\t\t\t\t\t\tTitre: ");
                scanf(" %[^\n]", livre.titre);
                printf("\t\t\t\t\t\tAuteur: ");
                scanf(" %[^\n]", livre.auteur);
                printf("\t\t\t\t\t\tISBN: ");
                scanf("%d", &livre.isbn);
                livre.disponible = true;
                ajouterLivre(&listeLivres, livre);
                 sauvegarderLivres(&listeLivres);
                printf("\t\t\t\t\t\tLe livre est ajoute.\n");

                break;
            }
            case 2: {
                int isbn;
                viderListe(&listeLivres);
                chargerLivres(&listeLivres);
                printf("\t\t\t\t\t\tISBN du livre a supprimer: ");
                scanf("%d", &isbn);
                supprimerLivre(&listeLivres, isbn);
                sauvegarderLivres(&listeLivres);
                printf("\t\t\t\t\t\tLe livre est supprime.\n");
                break;
            }
            case 3: {
                int isbn;
                viderListe(&listeLivres);
                chargerLivres(&listeLivres);
                printf("\t\t\t\t\t\tISBN du livre a emprunter: ");
                scanf("%d", &isbn);
                emprunterLivre(&listeLivres, &fileAttente, isbn);
                sauvegarderLivres(&listeLivres);
                break;
            }
            case 4: {
                int isbn;
                viderListe(&listeLivres);
                chargerLivres(&listeLivres);
                printf("\t\t\t\t\t\tISBN du livre a retourner: ");
                scanf("%d", &isbn);
                retournerLivre(&listeLivres, &fileAttente, isbn);
                 sauvegarderLivres(&listeLivres);
                break;
            }
            case 5: {
                printf("\t\t\t\t\t\tLA LISTES DES LIVRES:\n");
                afficherLivresDepuisFichier("livres.txt"); break;
            }


            case 0: {
                printf("\t\t\t\t\t\tAu revoir!\n");
                break;
            }
            default: {
                printf("\t\t\t\t\t\tChoix invalide. Veuillez reessayer.\n");
                break;
            }
        }
    } while (choix != 0);

    return 0;
}
