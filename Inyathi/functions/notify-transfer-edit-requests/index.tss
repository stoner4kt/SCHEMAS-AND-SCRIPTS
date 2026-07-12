import { serve } from 'https://deno.land/std@0.177.0/http/server.ts';

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
    'Access-Control-Allow-Headers': '*',
      'Access-Control-Allow-Methods': 'POST, OPTIONS',
      };

      serve(async (req: Request) => {
        if (req.method === 'OPTIONS') {
            return new Response('ok', { headers: corsHeaders });
              }

                if (req.method !== 'POST') {
                    return new Response(JSON.stringify({ error: 'Method not allowed' }), {
                          status: 405,
                                headers: { ...corsHeaders, 'Content-Type': 'application/json' },
                                    });
                                      }

                                        try {
                                            const {
                                                  transfer_recon_id,
                                                        driver_id,
                                                              driver_name,
                                                                    driver_email,
                                                                          driver_phone,
                                                                                reason,
                                                                                      week_start,
                                                                                            week_end,
                                                                                                  vehicle_reg,
                                                                                                      } = await req.json();

                                                                                                          if (!transfer_recon_id || !driver_id || !reason) {
                                                                                                                return new Response(JSON.stringify({ error: 'Missing required fields: transfer_recon_id, driver_id, reason' }), {
                                                                                                                        status: 400,
                                                                                                                                headers: { ...corsHeaders, 'Content-Type': 'application/json' },
                                                                                                                                      });
                                                                                                                                          }

                                                                                                                                              const resendApiKey = Deno.env.get('RESEND_APIKEY') ?? Deno.env.get('RESEND_API_KEY') ?? '';
                                                                                                                                                  if (!resendApiKey) {
                                                                                                                                                        return new Response(
                                                                                                                                                                JSON.stringify({ success: false, error: 'Resend API key is not configured' }),
                                                                                                                                                                        { status: 500, headers: { ...corsHeaders, 'Content-Type': 'application/json' } }
                                                                                                                                                                              );
                                                                                                                                                                                  }

                                                                                                                                                                                      const adminEmailRaw = Deno.env.get('ADMIN_EMAIL') ?? '';
                                                                                                                                                                                          const toEmails = adminEmailRaw.split(',').map((e) => e.trim()).filter(Boolean);
                                                                                                                                                                                              if (toEmails.length === 0) {
                                                                                                                                                                                                    toEmails.push('reeqieric41@gmail.com');
                                                                                                                                                                                                        }

                                                                                                                                                                                                            const fromEmail =
                                                                                                                                                                                                                  Deno.env.get('SENDER_EMAIL') ??
                                                                                                                                                                                                                        Deno.env.get('FROM_EMAIL') ??
                                                                                                                                                                                                                              'Inyathi Alerts <onboarding@resend.dev>';

                                                                                                                                                                                                                                  const timestamp = new Date().toLocaleString('en-ZA', {
                                                                                                                                                                                                                                        timeZone: 'Africa/Johannesburg',
                                                                                                                                                                                                                                              day: '2-digit',
                                                                                                                                                                                                                                                    month: 'short',
                                                                                                                                                                                                                                                          year: 'numeric',
                                                                                                                                                                                                                                                                hour: '2-digit',
                                                                                                                                                                                                                                                                      minute: '2-digit',
                                                                                                                                                                                                                                                                          });

                                                                                                                                                                                                                                                                              const htmlBody = `
                                                                                                                                                                                                                                                                              <!DOCTYPE html>
                                                                                                                                                                                                                                                                              <html>
                                                                                                                                                                                                                                                                              <head>
                                                                                                                                                                                                                                                                                <meta charset="utf-8">
                                                                                                                                                                                                                                                                                  <meta name="viewport" content="width=device-width, initial-scale=1.0">
                                                                                                                                                                                                                                                                                    <title>Transfer Recon Edit Request</title>
                                                                                                                                                                                                                                                                                    </head>
                                                                                                                                                                                                                                                                                    <body style="font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif; background-color: #f8fafc; color: #1e293b; padding: 24px 16px; margin: 0;">
                                                                                                                                                                                                                                                                                      <div style="max-width: 600px; margin: 0 auto; background-color: #ffffff; border-radius: 12px; border: 1px solid #e2e8f0; overflow: hidden; box-shadow: 0 4px 6px -1px rgba(0,0,0,0.05);">

                                                                                                                                                                                                                                                                                          <!-- Header Banner -->
                                                                                                                                                                                                                                                                                              <div style="background-color: #0f2744; color: #ffffff; padding: 24px; text-align: center;">
                                                                                                                                                                                                                                                                                                    <h1 style="margin: 0; font-size: 20px; font-weight: 800; text-transform: uppercase; letter-spacing: 0.05em;">✏️ Transfer Recon Edit Request</h1>
                                                                                                                                                                                                                                                                                                          <p style="margin: 8px 0 0 0; font-size: 14px; opacity: 0.8;">Inyathi Fleet Management System</p>
                                                                                                                                                                                                                                                                                                              </div>

                                                                                                                                                                                                                                                                                                                  <!-- Main Content -->
                                                                                                                                                                                                                                                                                                                      <div style="padding: 24px; line-height: 1.6;">

                                                                                                                                                                                                                                                                                                                            <!-- Action Required Banner -->
                                                                                                                                                                                                                                                                                                                                  <div style="background-color: #fef9c3; border: 1px solid #fde047; border-radius: 8px; padding: 14px 16px; margin-bottom: 24px; text-align: center;">
                                                                                                                                                                                                                                                                                                                                          <span style="display: block; font-size: 12px; font-weight: 800; color: #854d0e; text-transform: uppercase; letter-spacing: 0.08em;">Action Required</span>
                                                                                                                                                                                                                                                                                                                                                  <span style="display: block; font-size: 14px; color: #713f12; margin-top: 4px;">A driver has requested permission to edit a submitted Transfer Recon Sheet. Please review and approve or reject in the Admin Dashboard.</span>
                                                                                                                                                                                                                                                                                                                                                        </div>

                                                                                                                                                                                                                                                                                                                                                              <!-- Driver Info -->
                                                                                                                                                                                                                                                                                                                                                                    <h3 style="margin-top: 0; margin-bottom: 12px; font-size: 13px; font-weight: 800; text-transform: uppercase; letter-spacing: 0.05em; color: #64748b; border-bottom: 1px solid #f1f5f9; padding-bottom: 6px;">Driver Details</h3>
                                                                                                                                                                                                                                                                                                                                                                          <table style="width: 100%; border-collapse: collapse; margin-bottom: 24px; font-size: 13px;">
                                                                                                                                                                                                                                                                                                                                                                                  <tr>
                                                                                                                                                                                                                                                                                                                                                                                            <td style="padding: 7px 0; color: #64748b; font-weight: 600; width: 150px;">Driver Name:</td>
                                                                                                                                                                                                                                                                                                                                                                                                      <td style="padding: 7px 0; color: #0f172a; font-weight: 700;">${driver_name ?? 'N/A'}</td>
                                                                                                                                                                                                                                                                                                                                                                                                              </tr>
                                                                                                                                                                                                                                                                                                                                                                                           
