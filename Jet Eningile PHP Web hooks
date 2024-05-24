<?php
// webhook_listener.php

// Include WordPress functions to access WordPress functionalities within this script
require_once('/path/to/wordpress/wp-load.php');

/**
 * Function to update JetEngine meta field
 *
 * @param int $post_id The ID of the post to update.
 * @param string $meta_key The meta key to update.
 * @param mixed $value The value to set for the meta key.
 */
function update_jetengine_value($post_id, $meta_key, $value) {
    update_post_meta($post_id, $meta_key, $value);
}

// Get the raw POST data from the webhook
$input = file_get_contents('php://input');
$data = json_decode($input, true);

// Check the type of event received from Mux
if (isset($data['type']) && $data['type'] == 'video.asset.created') {
    // Extract the asset ID and playback ID from the webhook data
    $asset_id = $data['data']['id'];
    $playback_id = $data['data']['playback_ids'][0]['id']; // Adjust this based on the actual structure of the webhook data

    // Generate the HLS playback URL using the playback ID
    $playback_url = "https://stream.mux.com/{$playback_id}.m3u8";

    // Get the current time to find the closest scheduled stream
    $current_time = new DateTime();

    // Define the arguments to query JetEngine posts with the closest scheduled time
    $args = [
        'post_type' => 'your_custom_post_type', // Replace 'your_custom_post_type' with the actual post type used in JetEngine
        'meta_query' => [
            [
                'key' => 'scheduled_time', // Replace 'scheduled_time' with the actual meta key used to store the scheduled time
                'value' => $current_time->format('Y-m-d H:i:s'),
                'compare' => '<=', // Find posts with scheduled time less than or equal to the current time
                'type' => 'DATETIME'
            ]
        ],
        'orderby' => 'meta_value', // Order the results by the meta value (scheduled time)
        'order' => 'DESC', // Order the results in descending order to get the closest past scheduled time
        'posts_per_page' => 1 // Limit the results to one post
    ];

    // Perform the query to find the relevant post
    $query = new WP_Query($args);

    // Check if any posts were found
    if ($query->have_posts()) {
        // Get the ID of the found post
        $query->the_post();
        $post_id = get_the_ID();

        // Define the meta key for the JetEngine field to update
        $meta_key = 'jet_engine_meta_key'; // Replace 'jet_engine_meta_key' with the actual meta key used in JetEngine

        // Update the JetEngine meta field with the generated playback URL
        update_jetengine_value($post_id, $meta_key, $playback_url);

        // Optionally, save the asset ID in a custom field for reference
        update_post_meta($post_id, 'mux_asset_id', $asset_id);

        // Respond with a 200 status code indicating success
        http_response_code(200);
        echo json_encode(['status' => 'success']);
    } else {
        // Respond with a 404 status code if no matching post is found
        http_response_code(404);
        echo json_encode(['status' => 'no matching post found']);
    }

    // Reset post data to ensure the query does not interfere with other queries
    wp_reset_postdata();
} else {
    // Respond with a 400 status code for invalid requests or unsupported event types
    http_response_code(400);
    echo json_encode(['status' => 'invalid event type']);
}

?>
