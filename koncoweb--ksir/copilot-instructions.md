## ksir

> create table public.inventory (


create table public.inventory (
  id uuid not null default gen_random_uuid (),
  company_id uuid not null,
  product_id uuid not null,
  location_type character varying(20) not null,
  location_name character varying(255) not null,
  quantity integer not null default 0,
  min_stock integer null default 0,
  max_stock integer null,
  updated_at timestamp with time zone null default now(),
  variation_id uuid null,
  constraint inventory_pkey primary key (id),
  constraint inventory_company_id_fkey foreign KEY (company_id) references companies (id) on delete CASCADE,
  constraint inventory_product_id_fkey foreign KEY (product_id) references products (id) on delete CASCADE,
  constraint inventory_variation_id_fkey foreign KEY (variation_id) references product_variations (id) on delete CASCADE,
  constraint inventory_location_type_check check (
    (
      (location_type)::text = any (
        (
          array[
            'warehouse'::character varying,
            'store'::character varying
          ]
        )::text[]
      )
    )
  )
) TABLESPACE pg_default;

create index IF not exists idx_inventory_company_id on public.inventory using btree (company_id) TABLESPACE pg_default;

create index IF not exists idx_inventory_product_id on public.inventory using btree (product_id) TABLESPACE pg_default;

create index IF not exists idx_inventory_variation_id on public.inventory using btree (variation_id) TABLESPACE pg_default;

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/koncoweb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
