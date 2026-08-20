# Générer la vidéo photoréaliste — mode d'emploi

Le fichier `microphone-cinematic-animation.json` est la **spécification de la scène**.
Un navigateur ne peut pas la transformer en image photoréaliste : pour obtenir un rendu
« vrai footage », il faut passer par un générateur vidéo IA. Voici les prompts prêts à
coller, dérivés de la spec v3.1.

---

## 1. Prompt master (texte → vidéo)

> Static tripod shot on a 50mm full-frame cinema lens at T2.0. A premium matte-black
> studio condenser microphone on a small round desk stand, centered slightly below the
> middle of the frame, razor sharp, at the edge of a mirror-calm alpine lake at sunrise.
> Huge snow-capped mountains softly out of focus behind it, natural rock formations and
> atmospheric haze, thin morning fog drifting slowly over the water, golden sunrise light
> low on the horizon with cool blue shadows, subtle volumetric light rays through the
> mist, tiny dust motes catching the light. Distant pine trees barely moving in the
> morning wind. Extremely slow, almost imperceptible 1% push-in. Photorealistic, shot on
> a professional cinema camera, natural HDR exposure, soft filmic contrast, premium
> commercial color grading, slightly desaturated. Calm, luxurious, minimal.
> No text, no logos, no people, no animals.

**Negative prompt** (si le modèle le supporte) :
> illustration, 3D render, CGI, cartoon, low-poly, vector art, oversaturated, neon,
> excessive bloom, camera shake, fast motion, people, animals, text, watermark, logo

---

## 2. Boucle parfaite : la bonne technique

Les modèles vidéo ne bouclent pas naturellement. Deux méthodes fiables :

1. **Image fixe → vidéo avec première = dernière frame** (recommandé)
   - Génère d'abord une **image hero** photoréaliste de la scène (Midjourney, Imagen,
     Firefly…) avec le prompt ci-dessus adapté en photo.
   - Passe cette image en *image-to-video* avec la **même image en keyframe de début ET
     de fin** (Kling et Runway le permettent). Le clip revient exactement à son point de
     départ → boucle invisible.
2. **Luma Dream Machine** possède une option **Loop** native — le plus simple si tu y as
   accès.

À défaut, un fondu-enchaîné de 1 s entre la fin et le début dans un éditeur
(DaVinci Resolve, gratuit) donne une boucle très propre sur ce type de plan quasi fixe.

---

## 3. Réglages par modèle

| Modèle | Durée | Astuce |
|---|---|---|
| **Kling 2.x** (kling.ai) | 5 ou 10 s | Image-to-video avec start+end frame identiques → boucle parfaite. Mode « Professional ». |
| **Luma Dream Machine** | 5–9 s | Activer **Loop**. Enchaîner deux segments si besoin de 10 s. |
| **Runway Gen-4** | 5 ou 10 s | First/last frame identiques. Motion très faible (1–2). |
| **Google Veo 3.x** (Gemini) | 8 s | Pas de keyframe de fin → utiliser la méthode fondu. Excellent photoréalisme. |
| **Sora** | 10 s | Préciser « perfect seamless loop » dans le prompt + vérifier. |

**4K :** la plupart sortent en 720p–1080p. Upscale ensuite (Topaz Video AI, ou
l'upscaler intégré de Freepik/Magnific avec un compte premium) vers 3840×2160,
export H.265 ~28 Mbps.

---

## 4. L'onde vocale : à NE PAS mettre dans la vidéo

Ton app détecte le micro : l'onde doit réagir **en direct** au son réel, pas être un
enregistrement figé dans le footage. La bonne architecture :

- **Fond** : la vidéo photoréaliste en boucle (sans onde).
- **Par-dessus** : l'onde cyan rendue en temps réel par l'app.

C'est exactement ce que fait `app-background.html` dans ce dépôt : il lit
`background.mp4` en boucle plein écran et dessine l'onde par-dessus, branchée sur le
micro (avec repli sur un rythme de parole synthétique si l'accès micro est refusé).
Les couleurs, la position et le comportement de l'onde sont lus dans
`microphone-cinematic-animation.json`.

**Pipeline complet :**
1. Image hero (générateur d'images) → 2. Image-to-video bouclée (Kling/Luma/Runway)
→ 3. Upscale 4K → 4. Encoder en `background.mp4` → 5. Ouvrir `app-background.html`.
