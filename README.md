# python---- FILE: README.md ---

eCourts Listing Checker (Python Project)

Author: Shubhangi Singh
B.Tech CSE, 2nd Year
Email: chaudharyshubhangi57@gmail.com

Overview

This project simulates checking court case listings from the Indian eCourts system. It is designed for internship submission, running offline to avoid captcha and network issues.

Features

Search by CNR or Case Type + Number + Year

Check if a case is listed today or tomorrow

Display serial number and court name

Optionally simulate downloading case PDF

Optionally simulate downloading the entire cause list for today

Simple console output and JSON output saved to output/ folder


Installation

1. Ensure Python 3.8+ is installed.


2. Install dependencies:



pip install -r requirements.txt

Usage

# Check case by CNR today
python ecourts_scraper.py --cnr DLST020314162024 --today

# Check case by Case Type + Number + Year for tomorrow
python ecourts_scraper.py --case-type RC --case-num 123 --case-year 2024 --tomorrow

# Download cause list for today
python ecourts_scraper.py --causelist --today

Sample Output

Case found!
CNR: DLST020314162024
Listing Date: 15-Oct-2025
Status: Listed Today
Serial No: 5
Court: Sessions Court - Dwarka
Result saved to: output/case_result_DLST020314162024_2025-10-15.json
Simulated PDF downloaded to: output/DLST020314162024.pdf

--- FILE: requirements.txt --- argparse requests

--- FILE: ecourts_scraper.py ---

#!/usr/bin/env python3
"""
ecourts_scraper.py
Simulated eCourts Listing Checker for internship submission
Author: Shubhangi Singh (B.Tech CSE, 2nd Year)
"""

import argparse
import json
import os
from datetime import datetime, timedelta

# --- Simulated dataset -------------------------------------------------
SIM_DATA = {}
TODAY = datetime.now().date()
TOMORROW = TODAY + timedelta(days=1)

SIM_DATA[str(TODAY)] = [
    {"cnr": "DLST020314162024", "case_type": "RC", "case_num": "1620", "case_year": "2024",
     "serial_no": 5, "court_name": "Sessions Court - Dwarka", "pdf_url": "https://simulated.ecourts.gov.in/pdf/DLST020314162024.pdf"},
    {"cnr": "DLST020314162025", "case_type": "CIV", "case_num": "77", "case_year": "2025",
     "serial_no": 12, "court_name": "Civil Court - South West", "pdf_url": None}
]
SIM_DATA[str(TOMORROW)] = [
    {"cnr": "MHCC010100012024", "case_type": "CR", "case_num": "12", "case_year": "2024",
     "serial_no": 3, "court_name": "City Court - Pune", "pdf_url": "https://simulated.ecourts.gov.in/pdf/MHCC010100012024.pdf"}
]

# --- Helpers ------------------------------------------------------------
def ensure_output_dir(path): os.makedirs(path, exist_ok=True)

def save_json(obj, path):
    with open(path, "w", encoding="utf-8") as f:
        json.dump(obj, f, indent=2, ensure_ascii=False)

def fetch_causelist_for_date(target_date):
    return SIM_DATA.get(target_date, [])

def search_case_in_causelist(causelist, cnr=None, case_type=None, case_num=None, case_year=None):
    for entry in causelist:
        if cnr and entry.get("cnr") == cnr: return entry
        if case_type and case_num and case_year:
            if (entry.get("case_type") == case_type and
                str(entry.get("case_num")) == str(case_num) and
                str(entry.get("case_year")) == str(case_year)):
                return entry
    return None

def download_pdf_simulated(pdf_url, out_dir, filename=None):
    ensure_output_dir(out_dir)
    if not filename: filename = os.path.basename(pdf_url)
    path = os.path.join(out_dir, filename)
    with open(path, "wb") as f: f.write(b"%PDF-1.4\n% simulated pdf content")
    return path

# --- CLI ---------------------------------------------------------------
def parse_args():
    p = argparse.ArgumentParser(description="Simulated eCourts Listing Checker")
    group = p.add_mutually_exclusive_group(required=True)
    group.add_argument("--cnr", help="Full CNR number")
    group.add_argument("--case-type", help="Case type (RC/CIV/CR)")
    p.add_argument("--case-num", help="Case number")
    p.add_argument("--case-year", help="Case year")
    date_group = p.add_mutually_exclusive_group()
    date_group.add_argument("--today", action="store_true")
    date_group.add_argument("--tomorrow", action="store_true")
    p.add_argument("--causelist", action="store_true")
    p.add_argument("--download-pdf", action="store_true")
    p.add_argument("--out", default="output")
    return p.parse_args()

# --- Main --------------------------------------------------------------
def main():
    args = parse_args()
    out_dir = args.out
    ensure_output_dir(out_dir)

    target_date = str(TOMORROW) if args.tomorrow else str(TODAY)
    causelist = fetch_causelist_for_date(target_date)

    if args.causelist:
        out_path = os.path.join(out_dir, f"cause_list_{target_date}.json")
        save_json(causelist, out_path)
        print(f"Cause list for {target_date} saved to: {out_path}")
        print(f"Total entries: {len(causelist)}")

    match = None
    if args.cnr: match = search_case_in_causelist(causelist, cnr=args.cnr)
    else:
        if not (args.case_num and args.case_year):
            print("--case-num and --case-year required with --case-type"); return
        match = search_case_in_causelist(causelist, case_type=args.case_type, case_num=args.case_num, case_year=args.case_year)

    result_obj = {"query_date": target_date, "query": {"cnr": args.cnr, "case_type": args.case_type, "case_num": args.case_num, "case_year": args.case_year}, "found": False, "case": None}

    if match:
        result_obj["found"] = True
        result_obj["case"] = match
        print("\nCase found!")
        print(f"Serial No : {match.get('serial_no')}")
        print(f"Court Name : {match.get('court_name')}")
        print(f"CNR : {match.get('cnr')}")
        out_path = os.path.join(out_dir, f"case_result_{match.get('cnr')}_{target_date}.json")
        save_json(result_obj, out_path)
        print(f"Result saved to: {out_path}")
        if args.download_pdf and match.get("pdf_url"):
            pdf_path = download_pdf_simulated(match.get("pdf_url"), out_dir)
            print(f"Simulated PDF downloaded to: {pdf_path}")
        elif args.download_pdf:
            print("No PDF link available.")
    else:
        print("\nCase not listed on the requested date.")
        out_path = os.path.join(out_dir, f"case_result_not_found_{target_date}.json")
        save_json(result_obj, out_path)
        print(f"Result saved to: {out_path}")

if __name__ == "__main__": main()

