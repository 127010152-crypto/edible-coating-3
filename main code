"""
Fruit Edible Coating Database and Shelf-Life Recommendation System
-------------------------------------------------------------------
Uses ONE CSV file kept in the same folder as this program:
  fruit_coating_database.csv -> fruit info + one row per coating treatment
  (fruits with no research data yet have one row with "Data not available")

To add more fruits or research data, just add rows to the CSV
(or use the menu options inside the program). No code change needed.
"""

import csv
import re
import textwrap

DB_FILE = "fruit_coating_database.csv"
NA = "Not reported in source"   # used when a paper does not give a value

# column names (same order as the CSV header)
FRUIT_COLUMNS = ["fruit_id", "fruit_name", "scientific_name", "alt_names", "item_type",
                 "fruit_category", "climacteric_status", "perishability",
                 "postharvest_problems", "deterioration_causes"]

COATING_COLUMNS = ["coating_name", "coating_materials", "origin", "formulation",
                   "concentration", "application_method", "storage_temperature",
                   "storage_conditions", "control_shelf_life_days", "coated_shelf_life_days",
                   "additional_shelf_life_days", "shelf_life_improvement_percentage",
                   "weight_loss_effect", "firmness_effect", "antimicrobial_effect",
                   "antioxidant_effect", "ripening_effect", "limitations", "source_type",
                   "reference", "doi", "source_url"]

ALL_COLUMNS = FRUIT_COLUMNS + COATING_COLUMNS

# words that suggest a good result (used in the scoring, very simple method)
GOOD_WORDS = ["lower", "reduc", "delay", "maintain", "retain", "better", "slow",
              "best", "kept", "less", "preserv", "higher"]


# ---------------------------------------------------------------
# 1. Loading and saving the database
# ---------------------------------------------------------------
def load_csv(filename):
    """Read a CSV file and return a list of dictionaries (one per row)."""
    try:
        with open(filename, newline="", encoding="utf-8") as f:
            return list(csv.DictReader(f))
    except FileNotFoundError:
        print("File not found:", filename)
        return []


def split_database(rows):
    """Split the single table into a fruit list (one per fruit) and a treatment list."""
    fruits = []
    seen = set()
    for row in rows:
        if row["fruit_id"] not in seen:
            seen.add(row["fruit_id"])
            fruits.append(row)
    # a row without a coating name is just a fruit with no research data yet
    treatments = [r for r in rows if has_data(r["coating_name"])]
    return fruits, treatments


def append_row(filename, columns, row):
    """Add one new row at the end of a CSV file."""
    with open(filename, "a", newline="", encoding="utf-8") as f:
        writer = csv.DictWriter(f, fieldnames=columns)
        writer.writerow(row)


# ---------------------------------------------------------------
# 2. Searching
# ---------------------------------------------------------------
def find_fruit(name, fruits):
    """Case-insensitive search by common name, scientific name or alternative name."""
    name = name.strip().lower()
    for fruit in fruits:
        names = [fruit["fruit_name"].lower(), fruit["scientific_name"].lower()]
        names += [n.strip().lower() for n in fruit["alt_names"].split(";")]
        if name in names:
            return fruit
    return None


def get_coatings(fruit_id, coatings):
    """All coating treatments for one fruit."""
    return [c for c in coatings if c["fruit_id"] == fruit_id]


def to_number(text):
    """Pull the first number out of a text like '12' or '11 (up to...)'. Returns None if no number."""
    match = re.search(r"\d+\.?\d*", text or "")
    return float(match.group()) if match else None


def has_data(text):
    """True if the field actually has information (blank or 'not reported' = no data)."""
    if not text:
        return False
    text = text.strip()
    return text != "" and not text.lower().startswith(("not reported", "data not available"))


# ---------------------------------------------------------------
# 3. Displaying
# ---------------------------------------------------------------
def show_fruit_info(fruit):
    print("\nFruit:", fruit["fruit_name"])
    print("Scientific name:", fruit["scientific_name"])
    print("Type:", fruit["item_type"], "|", fruit["fruit_category"], "|", fruit["climacteric_status"])
    print("Perishability:", fruit["perishability"])
    print("Common problems:", fruit["postharvest_problems"])
    print("Causes of deterioration:", fruit["deterioration_causes"])


def show_treatments(treatments):
    print("\nAvailable edible coating treatments:")
    for i, t in enumerate(treatments, 1):
        print(f"\n{i}. {t['coating_name']}")
        print("   - Materials used:", t["coating_materials"], f"({t['origin']})")
        print("   - Coating type:", coating_type(t["coating_materials"]))
        print("   - Formulation:", t["formulation"])
        print("   - Concentration:", t["concentration"])
        print("   - Application method:", t["application_method"])
        print("   - Storage conditions:", t["storage_temperature"], "|", t["storage_conditions"])
        print("   - Shelf life without coating (days):", t["control_shelf_life_days"])
        print("   - Shelf life with coating (days):", t["coated_shelf_life_days"])
        print("   - Additional shelf life (days):", t["additional_shelf_life_days"])
        print("   - Improvement (%):", t["shelf_life_improvement_percentage"])
        print("   - Weight loss:", t["weight_loss_effect"])
        print("   - Firmness:", t["firmness_effect"])
        print("   - Antimicrobial:", t["antimicrobial_effect"])
        print("   - Antioxidant:", t["antioxidant_effect"])
        print("   - Ripening:", t["ripening_effect"])
        print("   - Limitations:", t["limitations"])
        print("   - Source type:", t["source_type"])
        print("   - Scientific source:", t["reference"])
        print("   - DOI:", t["doi"])
        print("   - Link:", t["source_url"])


def show_references(treatments):
    print("\nReferences:")
    for i, t in enumerate(treatments, 1):
        print(f"{i}. [{t['source_type']}] {t['reference']}")
        print(f"   DOI: {t['doi']}")
        print(f"   Link: {t['source_url']}")


def compare_side_by_side(treatments, width=26, per_block=3):
    """Print a table with the treatments as columns (3 at a time so it fits the screen)."""
    rows = [("Coating", "coating_name"), ("Materials", "coating_materials"),
            ("Concentration", "concentration"), ("Temperature", "storage_temperature"),
            ("Control (days)", "control_shelf_life_days"),
            ("Coated (days)", "coated_shelf_life_days"),
            ("Extra days", "additional_shelf_life_days"),
            ("Improvement %", "shelf_life_improvement_percentage"),
            ("Weight loss", "weight_loss_effect"), ("Firmness", "firmness_effect"),
            ("Antimicrobial", "antimicrobial_effect"), ("Antioxidant", "antioxidant_effect")]
    for start in range(0, len(treatments), per_block):
        block = treatments[start:start + per_block]
        print("\n" + "=" * (16 + (width + 3) * len(block)))
        for label, key in rows:
            # wrap long text so columns stay aligned
            cells = [textwrap.wrap(t[key] or NA, width) or [""] for t in block]
            height = max(len(c) for c in cells)
            for line in range(height):
                text = label if line == 0 else ""
                print(f"{text:<15}", end="")
                for c in cells:
                    print("| " + (c[line] if line < len(c) else "").ljust(width) + " ", end="")
                print()
            print("-" * (16 + (width + 3) * len(block)))


# ---------------------------------------------------------------
# 4. Highest extension and best recommendation
# ---------------------------------------------------------------
def highest_extension(treatments):
    """Find the coating with the highest REPORTED shelf-life extension (not the 'best' one)."""
    best = None
    best_value = None
    for t in treatments:
        days = to_number(t["additional_shelf_life_days"])
        if days is not None and (best_value is None or days > best_value):
            best, best_value = t, days

    print("\nHighest reported shelf-life extension:")
    if best:
        print(f"  {best['coating_name']} -> +{best_value:g} days "
              f"({best['storage_temperature']}, {best['storage_conditions']})")
    else:
        print("  No treatment has both control and coated shelf life reported,")
        print("  so the extension cannot be calculated.")
        # at least show the longest coated shelf life that was reported
        longest = None
        longest_value = None
        for t in treatments:
            days = to_number(t["coated_shelf_life_days"])
            if days is not None and (longest_value is None or days > longest_value):
                longest, longest_value = t, days
        if longest:
            print(f"  (Longest coated shelf life reported: {longest['coating_name']}, "
                  f"{longest_value:g} days - no control value to compare with)")


def score_treatment(t):
    """
    Give a treatment a simple score out of 100.
    Also returns how many performance fields have data, and the reasons.
    NOTE: this is a student-level scoring method using keywords. It is a guide only,
    always read the actual paper before using a coating.
    """
    score = 0
    reasons = []
    data_points = 0

    # shelf life improvement (max 25 points)
    pct = to_number(t["shelf_life_improvement_percentage"])
    if pct is not None:
        score += min(pct, 100) / 100 * 25
        data_points += 1
        reasons.append(f"shelf life improved by {pct:g}%")

    # quality effects (10 points if clearly positive, 5 if just reported)
    quality_fields = [("weight_loss_effect", "weight loss"), ("firmness_effect", "firmness"),
                      ("antimicrobial_effect", "antimicrobial effect"),
                      ("antioxidant_effect", "antioxidants"), ("ripening_effect", "ripening")]
    for field, label in quality_fields:
        text = t[field]
        if has_data(text):
            data_points += 1
            if any(word in text.lower() for word in GOOD_WORDS):
                score += 10
                reasons.append(f"good result for {label}")
            else:
                score += 5

    # safety of ingredients (natural origin is safer, synthetic gets no bonus)
    origin = t["origin"].lower()
    if origin.startswith("natural"):
        score += 10
        reasons.append("natural, edible ingredients")
    elif has_data(t["origin"]):
        score += 3

    # strength of evidence
    if t["source_type"].lower().startswith("experimental"):
        score += 10
        reasons.append("direct experimental evidence")
    elif t["source_type"].lower().startswith("review"):
        score += 5
    if has_data(t["doi"]):
        score += 5

    return score, data_points, reasons


def recommend_best(treatments, min_data_points=3):
    """Compare the coatings and print the best one, or say the evidence is insufficient."""
    # only treatments with enough data can be compared fairly
    good_enough = []
    for t in treatments:
        score, points, reasons = score_treatment(t)
        # a treatment must have a reported shelf life number AND enough other results
        has_shelf_life = (to_number(t["coated_shelf_life_days"]) is not None
                          or to_number(t["shelf_life_improvement_percentage"]) is not None)
        if points >= min_data_points and has_shelf_life:
            good_enough.append((score, t, reasons))

    print("\n" + "=" * 50)
    print("BEST RECOMMENDED EDIBLE COATING")
    print("=" * 50)

    if len(good_enough) < 2:
        print("Evidence is insufficient to confidently choose ONE best coating")
        print(f"(only {len(good_enough)} treatment(s) have enough reported results to compare).")
        # still show the best option we have, clearly marked as limited evidence
        scored = []
        for t in treatments:
            sc, points, reasons = score_treatment(t)
            scored.append((sc, t))
        scored.sort(key=lambda x: x[0], reverse=True)
        t = scored[0][1]
        print("\nBest AVAILABLE option (limited evidence, treat as a starting point):")
        print("  Coating name:", t["coating_name"])
        print("  Materials:", t["coating_materials"], "|", coating_type(t["coating_materials"]))
        print("  Concentration:", t["concentration"])
        print("  Storage:", t["storage_temperature"], "|", t["storage_conditions"])
        print("  Shelf life (days) coated / control:", t["coated_shelf_life_days"], "/",
              t["control_shelf_life_days"])
        print("  Reference:", t["reference"], "| DOI:", t["doi"])
        return None

    good_enough.sort(key=lambda x: x[0], reverse=True)
    score, t, reasons = good_enough[0]
    print("Coating name:", t["coating_name"])
    print("Materials:", t["coating_materials"])
    print("Formulation:", t["formulation"], "|", t["concentration"])
    print("Application method:", t["application_method"])
    print("Expected shelf life:", t["coated_shelf_life_days"], "days")
    print("Shelf life extension:", t["additional_shelf_life_days"], "days")
    print("Storage conditions:", t["storage_temperature"], "|", t["storage_conditions"])
    print("Reason for recommendation:", "; ".join(reasons))
    print("Score:", round(score, 1), "/ 100")
    print("Scientific reference:", t["reference"], "| DOI:", t["doi"])
    print("Limitations:", t["limitations"])

    # warn if the second one is very close
    if len(good_enough) > 1 and score - good_enough[1][0] < 5:
        print("\nNote: the next best coating "
              f"({good_enough[1][1]['coating_name']}) scored almost the same.")
    return t


# ---------------------------------------------------------------
# 5. Updating the database
# ---------------------------------------------------------------
def ask(prompt):
    """Ask the user for a value; empty answer becomes 'Data not available'."""
    value = input(prompt).strip()
    return value if value else NA


def add_fruit(fruits):
    print("\n--- Add a new fruit or vegetable ---")
    row = {col: NA for col in ALL_COLUMNS}
    row["fruit_id"] = "F" + str(len(fruits) + 1).zfill(3)
    for col in FRUIT_COLUMNS[1:]:
        row[col] = ask(col + ": ")
    append_row(DB_FILE, ALL_COLUMNS, row)
    fruits.append(row)
    print("Added with id", row["fruit_id"], "(no coating data yet)")


def add_treatment(fruits, treatments):
    print("\n--- Add a new coating treatment ---")
    fruit = find_fruit(input("Fruit name: "), fruits)
    if fruit is None:
        print("Fruit not found in database. Add the fruit first (menu option 3).")
        return
    row = {col: fruit[col] for col in FRUIT_COLUMNS}
    skip = ["additional_shelf_life_days", "shelf_life_improvement_percentage"]
    for col in COATING_COLUMNS:
        if col not in skip:
            row[col] = ask(col + " (Enter to skip): ")

    # calculate extra days and % automatically if both shelf lives are numbers
    control = to_number(row["control_shelf_life_days"])
    coated = to_number(row["coated_shelf_life_days"])
    if control and coated:
        row["additional_shelf_life_days"] = str(round(coated - control, 1))
        row["shelf_life_improvement_percentage"] = str(round((coated - control) / control * 100, 1))
    else:
        row["additional_shelf_life_days"] = NA
        row["shelf_life_improvement_percentage"] = NA

    append_row(DB_FILE, ALL_COLUMNS, row)
    treatments.append(row)
    print("Treatment saved.")


# ---------------------------------------------------------------
# 5b. Coating type and suggestions for ANY fruit / vegetable
# ---------------------------------------------------------------
def coating_type(materials):
    """Classify a coating from its materials (polysaccharide / protein / lipid / composite)."""
    m = (materials or "").lower()
    groups = []
    if any(w in m for w in ["chitosan", "chitin", "pectin", "starch", "cellulose", "cmc",
                            "alginate", "gum", "aloe", "carrageenan", "mucilage", "seaweed"]):
        groups.append("polysaccharide")
    if any(w in m for w in ["gelatin", "whey", "zein", "sericin", "casein", "protein"]):
        groups.append("protein")
    if any(w in m for w in ["wax", "oil", "shellac", "paraffin", "lipid"]):
        groups.append("lipid/wax/resin")
    if not groups:
        return "Not classified"
    if len(groups) > 1:
        return "Composite (" + " + ".join(groups) + ")"
    return groups[0].capitalize() + "-based"


CITRUS = ["orange", "lemon", "mandarin", "pomelo", "grapefruit", "sweet lime", "kumquat"]
SOFT_FRUITS = ["strawberry", "blueberry", "raspberry", "blackberry", "mulberry", "cranberry",
               "grape", "sweet cherry"]
FRUIT_VEGETABLES = ["tomato", "cucumber", "bell pepper", "chilli", "brinjal", "okra", "zucchini",
                    "pumpkin", "bitter gourd", "bottle gourd", "green beans"]
LEAFY_OTHER = ["spinach", "lettuce", "broccoli", "cauliflower", "cabbage", "green peas",
               "sweet corn"]
ROOTS = ["potato", "sweet potato", "onion", "garlic", "carrot"]


def general_guidance(fruit):
    """
    Rule-based starting suggestion for ANY crop, based on its type.
    This is general guidance from the studies in this database and review papers.
    It is NOT an experiment on that exact crop, so confidence is lower.
    """
    name = fruit["fruit_name"].lower()
    climacteric = fruit["climacteric_status"].lower().startswith("climacteric")

    if name in CITRUS:
        return {"material": "Chitosan + carboxymethyl cellulose (CMC) bilayer, or pectin + beeswax with a natural antifungal oil (e.g. eugenol)",
                "concentration": "About 1-2% chitosan (typical range); confirm in the cited papers",
                "method": "Dipping for 1-2 min then air-dry (or spraying/brushing for large batches)",
                "storage": "Cool storage; avoid temperatures that cause chilling injury",
                "basis": "Review-level information (Pham et al. 2023, citing Arnon et al. 2014 and Alvarez et al.)",
                "confidence": "Low to moderate"}
    if name in SOFT_FRUITS:
        return {"material": "Chitosan with a plant essential oil or extract (antifungal, for grey mould)",
                "concentration": "About 1-1.5% chitosan (as in the strawberry studies in this database)",
                "method": "Gentle spraying or a brief dip, then air-dry (soft fruit bruises easily)",
                "storage": "About 4 °C",
                "basis": "Strawberry studies in this database (e.g. 1.5% chitosan; chitosan + propolis; chitosan + mango seed starch + orange peel oil)",
                "confidence": "Moderate (for strawberry); low for other berries"}
    if name in FRUIT_VEGETABLES:
        return {"material": "Aloe vera gel + chitosan (composite, polysaccharide-based)",
                "concentration": "Aloe vera gel 50-60% + chitosan 2% (best result in the tomato study)",
                "method": "Dipping, then air-dry",
                "storage": "About 8 °C (tomato study); chilling-sensitive crops should not go colder",
                "basis": "Tomato studies in this database (Int J Food Properties 2023, DOI 10.1080/10942912.2023.2263661)",
                "confidence": "Moderate for tomato; low for other crops in this group"}
    if name in LEAFY_OTHER:
        return {"material": "Chitosan (about 1%) or alginate-based coating",
                "concentration": "About 1% (typical starting point)",
                "method": "Gentle spraying or a quick dip, then air-dry",
                "storage": "Cold storage",
                "basis": "General coating knowledge; no study for this crop in this database",
                "confidence": "Low"}
    if name in ROOTS:
        return {"material": "Lipid/wax-based moisture-barrier coating, or starch/CMC coating (also helps limit weight loss)",
                "concentration": "Data not available in this database",
                "method": "Dipping or spraying, then air-dry",
                "storage": "Cool, dark, ventilated storage",
                "basis": "General coating knowledge; no study for this crop in this database",
                "confidence": "Low"}
    if climacteric:
        return {"material": "Chitosan (natural, antifungal) or aloe vera gel; chitosan nanoparticles or an essential oil can add effect",
                "concentration": "About 1-2% chitosan, or 25-50% aloe vera gel (ranges from the mango, papaya, banana and guava studies)",
                "method": "Dipping for 1-2 min then air-dry (simplest and most used)",
                "storage": "Cool storage above the chilling limit of the fruit",
                "basis": "Mango, banana, papaya and guava studies in this database",
                "confidence": "Moderate"}
    return {"material": "Chitosan, with or without aloe vera gel (the most widely studied natural coating)",
            "concentration": "About 1-2% chitosan",
            "method": "Dipping, then air-dry",
            "storage": "Cool storage",
            "basis": "General coating knowledge; no study for this crop in this database",
            "confidence": "Low"}


def show_suggestion(fruit, best):
    """Final summary: suggested coating material and method."""
    g = general_guidance(fruit)
    print("\n" + "=" * 50)
    print("SUGGESTED COATING MATERIAL AND METHOD")
    print("=" * 50)
    if best is not None:
        print("Evidence-based pick from the database:")
        print("  Material:", best["coating_materials"], "|", coating_type(best["coating_materials"]))
        print("  Concentration:", best["concentration"])
        print("  Method:", best["application_method"])
        print("  Reference:", best["reference"])
        print()
    print("General guidance for", fruit["fruit_name"], "(rule-based, not an experiment on this exact crop):")
    print("  Suggested material:", g["material"])
    print("  Coating type:", coating_type(g["material"]))
    print("  Suggested concentration:", g["concentration"])
    print("  Suggested method:", g["method"])
    print("  Storage tip:", g["storage"])
    print("  Based on:", g["basis"])
    print("  Confidence:", g["confidence"])
    print("  Always test on a small batch first and check food-safety rules.")


# ---------------------------------------------------------------
# 6. Menus
# ---------------------------------------------------------------
def fruit_menu(fruit, coatings):
    """Show the full report straight away, then a small menu for extras."""
    show_fruit_info(fruit)
    treatments = get_coatings(fruit["fruit_id"], coatings)
    best = None
    if treatments:
        show_treatments(treatments)
        highest_extension(treatments)
        best = recommend_best(treatments)
    else:
        print("\nNo research studies added for this fruit yet (use menu option 4 to add one).")
    show_suggestion(fruit, best)

    while treatments:
        print("\n1. Compare side by side   2. References   3. Back")
        choice = input("Choose: ").strip()
        if choice == "1":
            compare_side_by_side(treatments)
        elif choice == "2":
            show_references(treatments)
        elif choice == "3":
            break
        else:
            print("Please enter 1-3.")


def main():
    fruits, coatings = split_database(load_csv(DB_FILE))
    print("Fruit Edible Coating Decision-Support System")
    print(f"{len(fruits)} fruits/vegetables and {len(coatings)} coating treatments loaded.")

    while True:
        print("\n1. Search fruit   2. List fruits   3. Add fruit   4. Add coating data   5. Quit")
        choice = input("Choose: ").strip()
        if choice == "1":
            fruit = find_fruit(input("Enter fruit name: "), fruits)
            if fruit is None:
                print("Fruit not found in database")
            else:
                fruit_menu(fruit, coatings)
        elif choice == "2":
            for f in fruits:
                n = len(get_coatings(f["fruit_id"], coatings))
                print(f"{f['fruit_name']} ({f['scientific_name']}) - {n} treatment(s)")
        elif choice == "3":
            add_fruit(fruits)
        elif choice == "4":
            add_treatment(fruits, coatings)
        elif choice == "5":
            print("Bye!")
            break
        else:
            print("Please enter 1-5.")


if __name__ == "__main__":
    main()
