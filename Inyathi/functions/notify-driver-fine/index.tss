import { serve } from 'https://deno.land/std@0.177.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2';

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
      'Access-Control-Allow-Methods': 'POST, OPTIONS',
      };

      function jsonResponse(body: Record<string, unknown>, status = 200) {
        return new Response(JSON.stringify(body), {
            status,
                headers: { ...corsHeaders, 'Content-Type': 'application/json' },
                  });
                  }

                  function escapeHtml(value: unknown): string {
                    return String(value ?? '')
                        .replaceAll('&', '&amp;')
                            .replaceAll('<', '&lt;')
                                .replaceAll('>', '&gt;')
                                    .replaceAll('"', '&quot;')
                                        .replaceAll("'", '&#039;');
                                        }

                                        function formatCurrency(amount: unknown): string {
                                          if (amount === null || amount === undefined || amount === '') return 'Not specified';
                                            const numeric = Number(amount);
                                              return Number.isFinite(numeric) ? `R ${numeric.toFixed(2)}` : escapeHtml(amount);
                                              }

                                              serve(async (req: Request) => {
                                                if (req.method === 'OPTIONS') return new Response('ok', { headers: corsHeaders });
                                                  if (req.method !== 'POST') return jsonResponse({ error: 'Method not allowed' }, 405);

                                                    try {
                                                        const authHeader = req.headers.get('Authorization') ?? '';
                                                            const token = authHeader.replace(/^Bearer\s+/i, '');
                                                                if (!token) return jsonResponse({ error: 'Missing bearer token' }, 401);

                                                                    const { traffic_fine_id } = await req.json();
                                                                        if (!traffic_fine_id) return jsonResponse({ error: 'traffic_fine_id required' }, 400);

                                                                            const supabaseUrl = Deno.env.get('SUPABASE_URL') ?? '';
                                                                                const serviceRoleKey = Deno.env.get('SUPABASE_SERVICE_ROLE_KEY') ?? '';
                                                                                    const resendKey = Deno.env.get('RESEND_API_KEY') ?? '';

                                                                                        const supabaseAdmin = createClient(supabaseUrl, serviceRoleKey);
                                                                                            const { data: userResult, error: userError } = await supabaseAdmin.auth.getUser(token);
                                                                                                if (userError || !userResult.user) return jsonResponse({ error: 'Invalid bearer token' }, 401);

                                                                                                    const { data: requester, error: requesterError } = await supabaseAdmin
                                                                                                          .from('profiles')
                                                                                                                .select('id, role')
                                                                                                                      .eq('id', userResult.user.id)
                                                                                                                            .single();

                                                                                                                                if (requesterError || requester?.role !== 'admin') {
                                                                                                                                      return jsonResponse({ error: 'Admin access required' }, 403);
                                                                                                                                          }

                                                                                                                                              // Fetch the fine on its own — no joins to avoid FK requirement errors
                                                                                                                                                  const { data: fine, error: fineError } = await supabaseAdmin
                                                                                                                                                        .from('traffic_fines')
                                                                                                                                                              .select(`
                                                                                                                                                                      id,
                                                                                                                                                                              booking_id,
                                                                                                                                                                                      vehicle_reg,
                                                                                                                                                                                              driver_id,
                                                                                                                                                                                                      fine_timestamp,
                                                                                                                                                                                                              fine_reference,
                                                                                                                                                                                                                      location,
                                                                                                                                                                                                                              description,
                                                                                                                                                                                                                                      amount,
                                                                                                                                                                                                                                              notification_email
                                                                                                                                                                                                                                                    `)
                                                                                                                                                                                                                                                          .eq('id', traffic_fine_id)
                                                                                                                                                                                                                                                                .single();

                                                                                                                                                                                                                                                                    if (fineError || !fine) return jsonResponse({ error: 'Traffic fine not found' }, 404);

                                                                                                                                                                                                                                                                        // Separate lookup for driver profile (driver_id is a text key, not a UUID FK)
                                                                                                                                                                                                                                                                            const { data: driverProfile } = await supabaseAdmin
                                                                                                                                                                                                                                                                                  .from('profiles')
                                                                                                                                                                                                                                                                                        .select('name, phone, email')
                                                                                                                                                                                                                                                                                              .eq('driver_id', fine.driver_id)
                                                                                                                                                                                                                                                                                                    .single();

                                                                                                                                                                                                                                                                                                        // Separate lookup for booking details (only if booking_id exists)
                                                                                                                                                                                                                                                                                                            const { data: booking } = fine.booking_id
                                                                                                                                                                                                                                                                                                                  ? await supabaseAdmin
                                                                                                                                                                                                                                                                                                                            .from('bookings')
                                                                                                                                                                                                                                                                                                                                      .select('invoice_no, client_name, route')
                                                                                                                                                                                                                                                                                                                                                .eq('id', fine.booking_id)
                                                                                                                                                                                                                                                                                                                                                          .single()
                                                                                                                                                                                                                                                                                                                                                                : { data: null };

                                                                                                                                                                                                                                                                                                                                                                    const driverEmail = driverProfile?.email?.trim();
                                                                                                                                                                                                                                                                                                                                                                        const extraEmail = fine.notification_email?.trim();
                                                                                                                                                                                                                                                                                                                                                                            const recipients = Array.from(new Set([driverEmail, extraEmail].filter(Boolean)));

                                                                                                                                                                                                                                                                                                                                                                                if (!recipients.length) {
                                                                                                                                                                                                                                                                                                                                                                                      await supabaseAdmin
                                                                                                                                                                                                                                                                                                                                                                                              .from('traffic_fines')
                                                                                                                                                                                                                                                                                                                                                                                                      .update({ notification_error: 'No profile or notification email available' })
                                                                                                                                                                                                                                                                                                                                                                                                              .eq('id', traffic_fine_id);
                                                                                                                                                                                                                                                                                                                                                                                                                    return jsonResponse({ error: 'At least one recipient email is required' }, 400);
                                                                                                                                                                                                                                                                                                                                                                                                                        }

                                                                                                                                                                                                                                                                                                                                                                                                                            if (!resendKey) {
                                                                                                                                                                                                                                                                                                                                                                                                                                  await supabaseAdmin
                                                                                                                                                                                                                                                                                                                                                                                                                                          .from('traffic_fines')
                                                                                                                                                                                                                                                                                                                                                                                                                                                  .update({ notification_error: 'RESEND_API_KEY is not configured' })
                                                                                                                                                                                                                                                                                                                                                                                                                     
