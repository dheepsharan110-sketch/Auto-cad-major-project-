# Auto-cad-major-project-
Project on Auto cad 
from pathlib import Path

code = r'''; Conceptual 3D commercial building model for AutoCAD.
; APPLOAD this file, then type COMMERCIAL3D.
; Millimetres. Placeholder geometry only; NOT FOR CONSTRUCTION.

(defun cb-layer (nm col)
  (command "_.-LAYER" "_M" nm "_C" col nm "")
)
(defun cb-box (x1 y1 z1 x2 y2 z2)
  (command "_.BOX" (list x1 y1 z1) (list x2 y2 z1) (- z2 z1))
)
(defun cb-note (pt ht msg)
  (command "_.TEXT" "_J" "_MC" pt ht 0 msg)
)

(defun c:COMMERCIAL3D (/ oldos oldecho xs ys floors i j k x y z
                         colhalf beamhalf beamdepth slabthk floorh)
  (setq oldos (getvar "OSMODE") oldecho (getvar "CMDECHO"))
  (setvar "OSMODE" 0)
  (setvar "CMDECHO" 0)
  (setvar "INSUNITS" 4)
  (setq xs '(0 6000 12000 18000 24000))
  (setq ys '(0 6000 12000 18000))
  (setq floors '(0 3600 7200 10800))
  ; Illustrative dimensions only - replace with engineer-approved design.
  (setq colhalf 225 beamhalf 150 beamdepth 500 slabthk 150 floorh 3600)

  (cb-layer "COLUMNS" "1")
  (cb-layer "BEAMS" "4")
  (cb-layer "SLABS" "8")
  (cb-layer "STAIRCASE" "6")
  (cb-layer "GRID" "3")
  (cb-layer "ANNOTATION" "2")

  ; Columns from ground to roof.
  (setvar "CLAYER" "COLUMNS")
  (setq k 0)
  (while (< k 3)
    (setq z (nth k floors))
    (foreach x xs
      (foreach y ys
        (cb-box (- x colhalf) (- y colhalf) z
                (+ x colhalf) (+ y colhalf) (+ z floorh))
      )
    )
    (setq k (1+ k))
  )

  ; Beams at each elevated floor, in both grid directions.
  (setvar "CLAYER" "BEAMS")
  (foreach z '(3600 7200 10800)
    (setq j 0)
    (while (< j (length ys))
      (setq y (nth j ys) i 0)
      (while (< i 4)
        (setq x (nth i xs))
        (cb-box (+ x colhalf) (- y beamhalf) (- z beamdepth)
                (- (nth (1+ i) xs) colhalf) (+ y beamhalf) z)
        (setq i (1+ i))
      )
      (setq j (1+ j))
    )
    (setq i 0)
    (while (< i (length xs))
      (setq x (nth i xs) j 0)
      (while (< j 3)
        (setq y (nth j ys))
        (cb-box (- x beamhalf) (+ y colhalf) (- z beamdepth)
                (+ x beamhalf) (- (nth (1+ j) ys) colhalf) z)
        (setq j (1+ j))
      )
      (setq i (1+ i))
    )
  )

  ; Floor plates.
  (setvar "CLAYER" "SLABS")
  (foreach z '(3600 7200 10800)
    (setq i 0)
    (while (< i 4)
      (setq x (nth i xs) j 0)
      (while (< j 3)
        (setq y (nth j ys))
        (cb-box x y z (nth (1+ i) xs) (nth (1+ j) ys) (+ z slabthk))
        (setq j (1+ j))
      )
      (setq i (1+ i))
    )
  )

  ; Schematic stair steps in one corner bay at each storey.
  (setvar "CLAYER" "STAIRCASE")
  (foreach z '(0 3600 7200)
    (setq k 0)
    (while (< k 16)
      (cb-box 19000 (+ 300 (* k 250)) (+ z (* k 225))
              20200 (+ 300 (* (1+ k) 250)) (+ z (* (1+ k) 225)))
      (setq k (1+ k))
    )
  )

  ; Ground-level grid reference lines.
  (setvar "CLAYER" "GRID")
  (foreach x xs
    (command "_.LINE" (list x -1500 0) (list x 19500 0) "")
  )
  (foreach y ys
    (command "_.LINE" (list -1500 y 0) (list 25500 y 0) "")
  )

  ; Notes.
  (setvar "CLAYER" "ANNOTATION")
  (cb-note (list 12000 -2500 11500) 400 "3D COMMERCIAL BUILDING - CONCEPT MODEL")
  (cb-note (list 12000 -2500 11000) 250 "4 X 3 GRID BAYS | THREE UPPER FLOOR LEVELS")
  (cb-note (list 12000 -2500 10500) 200 "ALL DIMENSIONS IN mm | PLACEHOLDER GEOMETRY")
  (cb-note (list 12000 -2500 10000) 200 "NOT FOR CONSTRUCTION OR STRUCTURAL DESIGN")

  (command "_.ZOOM" "_E")
  (setvar "CMDECHO" oldecho)
  (setvar "OSMODE" oldos)
  (princ "\nConceptual 3D building created. Use 3DORBIT to inspect.")
  (princ "\nStructural analysis requires a separate validated analytical model.")
  (princ)
)
(princ "\nLoaded. Type COMMERCIAL3D to create the model.")
(princ)
'''
p = Path("/mnt/data/commercial_building_3d.lsp")
p.write_text(code, encoding="utf-8")
print(f"Created: {p}")
