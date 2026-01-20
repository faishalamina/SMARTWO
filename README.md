# SMARTWO
Membuat Web untuk Smart WO
import { createClient, SupabaseClient } from '@supabase/supabase-js';

let _supabaseAdmin: SupabaseClient | null = null;

/**
 * Returns a Supabase client using the SERVICE_ROLE_KEY.
 * This client bypasses RLS so use it only in server-side trusted code.
 */
export function getSupabaseServerClient(): SupabaseClient {
  if (_supabaseAdmin) return _supabaseAdmin;

  const supabaseUrl = process.env.NEXT_PUBLIC_SUPABASE_URL;
  const supabaseServiceRoleKey = process.env.SUPABASE_SERVICE_ROLE_KEY;

  if (!supabaseUrl || !supabaseServiceRoleKey) {
    throw new Error('Missing SUPABASE environment variables (NEXT_PUBLIC_SUPABASE_URL or SUPABASE_SERVICE_ROLE_KEY).');
  }

  _supabaseAdmin = createClient(supabaseUrl, supabaseServiceRoleKey, {
    auth: {
      // disable persisting to localStorage / cookies
      persistSession: false,
      // set global headers if needed
    },
    global: {
      // keep default fetch etc.
    },
  });

  return _supabaseAdmin;
}

/**
 * Helper to resolve user from a Bearer token using the Admin client.
 * Expects an access_token (JWT) obtained from client Supabase auth.
 */
export async function getUserFromBearerToken(bearerToken: string) {
  if (!bearerToken) return { user: null, error: { message: 'No token provided' } };

  const supabase = getSupabaseServerClient();
  try {
    // Supabase JS (v2) supports auth.getUser with a token argument
    const { data, error } = await supabase.auth.getUser(bearerToken);
    if (error) return { user: null, error };
    return { user: data.user, error: null };
  } catch (err: any) {
    return { user: null, error: err };
  }
}


\
