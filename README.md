function SynthesizerApp211
    % Create the UI figure
    fig = uifigure('Position', [100 100 800 600], 'Name', 'Synthesizer App');

    % Tab group for waveforms
    tg = uitabgroup(fig, 'Position', [50 50 700 300]);
    tabOriginal = uitab(tg, 'Title', 'Original Note Waveform');
    tabFSK = uitab(tg, 'Title', 'FSK Waveform');
    tabSpectrum = uitab(tg, 'Title', 'Frequency Spectrum');
    
    % Axes for waveforms
    axOriginal = uiaxes(tabOriginal, 'Position', [10 10 680 260]);
    title(axOriginal, 'Original Note Waveform');
    xlabel(axOriginal, 'Time (s)');
    ylabel(axOriginal, 'Amplitude');

    axFSK = uiaxes(tabFSK, 'Position', [10 10 680 260]);
    title(axFSK, 'FSK Waveform');
    xlabel(axFSK, 'Time (s)');
    ylabel(axFSK, 'Amplitude');

    axSpectrum = uiaxes(tabSpectrum, 'Position', [10 10 680 260]);
    title(axSpectrum, 'Frequency Spectrum');
    xlabel(axSpectrum, 'Frequency (Hz)');
    ylabel(axSpectrum, 'Magnitude');

    % UI Components
    % Dropdown for notes
    lblNote = uilabel(fig, 'Position', [50 500 100 30], 'Text', 'Select Note:');
    ddNote = uidropdown(fig, 'Position', [150 500 100 30], 'Items', {'C','C#','D', 'D#', 'E', 'F','F#', 'G','G#','A','A#', 'B'});

    % Slider for duration
    lblDuration = uilabel(fig, 'Position', [50 450 100 30], 'Text', 'Duration (s):');
    sldDuration = uislider(fig, 'Position', [150 460 100 3], 'Limits', [0.1 5], 'Value', 1);

    % Button to generate sound
    btnGenerate = uibutton(fig, 'Position', [50 400 100 30], 'Text', 'Generate Sound', ...
        'ButtonPushedFcn', @(btn,event) generateSound(ddNote.Value, sldDuration.Value, axOriginal, axSpectrum));

    % Button for FSK Modulation
    lblData = uilabel(fig, 'Position', [300 500 100 30], 'Text', 'Data Bits (0/1):');
    txtData = uitextarea(fig, 'Position', [400 500 100 30], 'Value', '101010');
    
    lblF1 = uilabel(fig, 'Position', [300 450 100 30], 'Text', 'Frequency 1:');
    txtF1 = uitextarea(fig, 'Position', [400 450 100 30], 'Value', '440');
    
    lblF2 = uilabel(fig, 'Position', [300 400 100 30], 'Text', 'Frequency 2:');
    txtF2 = uitextarea(fig, 'Position', [400 400 100 30], 'Value', '880');
    
    % Set the callback function for the dropdown after creating the text areas
    ddNote.ValueChangedFcn = @(dd,event) updateFrequencies(dd.Value, txtF1, txtF2);
    
    btnFSK = uibutton(fig, 'Position', [300 350 100 30], 'Text', 'FSK Modulate', ...
        'ButtonPushedFcn', @(btn,event) FSKModulateSound(txtData.Value, str2double(txtF1.Value), str2double(txtF2.Value), sldDuration.Value, axFSK, axSpectrum));
    
    % Button to play "Never Gonna Give You Up"
    btnPlayNGGYU = uibutton(fig, 'Position', [570 500 200 30], 'Text', 'Play Never Gonna Give You Up', ...
        'ButtonPushedFcn', @(btn,event) playNGGYU(axOriginal, axFSK, axSpectrum, sldDuration.Value));

    % Button to play FSK modulated "Never Gonna Give You Up"
    btnPlayNGGYUFSK = uibutton(fig, 'Position', [570 430 200 30], 'Text', 'Play NGGYU FSK', ...
        'ButtonPushedFcn', @(btn,event) playNGGYUFSK(axFSK, axSpectrum, sldDuration.Value));
end

% Function to update frequencies based on the selected note
function updateFrequencies(note, txtF1, txtF2)
    frequency = getNoteFrequency(note);
    harmonicFrequency = frequency * 2; % One octave higher for the harmonic
    txtF1.Value = num2str(frequency);
    txtF2.Value = num2str(harmonicFrequency);
end

% Function to generate sound for a given note and duration
function generateSound(note, duration, axOriginal, axSpectrum)
    fs = 44100; % Sample rate
    frequency = getNoteFrequency(note);
    y = generateHarmonics(frequency, duration, fs);
    plotWaveform(axOriginal, y, fs);
    plotSpectrum(axSpectrum, y, fs);
    sound(y, fs);
end

% Function to get the frequency of a note
function frequency = getNoteFrequency(note)
    switch note
        case 'C'
            frequency = 261.63;
        case 'C#'
            frequency = 277.18;
        case 'D'
            frequency = 293.66;
        case 'D#'
            frequency = 311.13;
        case 'E'
            frequency = 329.63;
        case 'F'
            frequency = 349.23;
        case 'F#'
            frequency = 369.99;
        case 'G'
            frequency = 392;
        case 'G#'
            frequency = 415.3;
        case 'A'
            frequency = 440;
        case 'A#'
            frequency = 466.16;
        case 'B'
            frequency = 493.88;
        otherwise
            frequency = 440; % Default to A if unknown
    end
end

% Function to generate harmonics
function y = generateHarmonics(frequency, duration, fs)
    t = 0:1/fs:duration;
    y = sin(2*pi*frequency*t);
    for n = 2:8
        if n ~= 7 % Skipping the 7th harmonic
            y = y + (1/n) * sin(2*pi*n*frequency*t);
        end
    end
end

% Function for FSK modulation
function FSKModulateSound(dataStr, f1, f2, bit_duration, axFSK, axSpectrum)
    fs = 44100; % Sample rate
    data = arrayfun(@(c) str2double(c), dataStr);
    
    % Generate FSK modulated signal
    yFSK = FSKModulate(data, f1, f2, bit_duration, fs);
    
    % Plot waveforms
    plotWaveform(axFSK, yFSK, fs);
    plotSpectrum(axSpectrum, yFSK, fs);
    
    % Play the FSK modulated sound
    sound(yFSK, fs);
end

% Function to perform FSK modulation
function y = FSKModulate(data, f1, f2, bit_duration, fs)
    t = 0:1/fs:bit_duration - 1/fs;
    y = [];
    for bit = data
        if bit == 0
            y = [y, sin(2*pi*f1*t)];
        else
            y = [y, sin(2*pi*f2*t)];
        end
    end
end

% Function to plot the waveform
function plotWaveform(ax, y, fs)
    t = (0:length(y)-1) / fs;
    plot(ax, t, y);
    ax.XLim = [0 max(t)];
    ax.YLim = [-1 1];
end

% Function to plot the frequency spectrum
function plotSpectrum(ax, y, fs)
    n = length(y);
    f = (0:n-1)*(fs/n);
    magnitude = abs(fft(y))/n;
    plot(ax, f, magnitude);
    ax.XLim = [0 fs/2];
    ax.YLim = [0 max(magnitude)];
end

% Function to play "Never Gonna Give You Up"
function playNGGYU(axOriginal, axFSK, axSpectrum, note_duration)
    fs = 44100; % Sample rate
    % Notes and durations based on the piano chords for "Never Gonna Give You Up"
    notes = {'C', 'D', 'F', 'D', 'A', 'A', 'G', 'C', 'D', 'F', 'D', 'G', ...
             'G', 'F', 'C', 'D', 'F', 'D', 'F', 'G', 'E', 'D', 'C', 'C', ...
             'G', 'F', 'C', 'D', 'F', 'D', 'A', 'A', 'G', 'C', 'D', 'F', ...
             'D', 'G', 'G', 'F', 'C', 'D', 'F', 'D', 'F', 'G', 'E', 'D', ...
             'C', 'C', 'G', 'F',};
    durations = [note_duration * ones(1, length(notes))];

    % Generate and play the melody
    melody = [];
    for i = 1:length(notes)
        note = notes{i};
        duration = durations(i);
        frequency = getNoteFrequency(note);
        y = generateHarmonics(frequency, duration, fs);
        melody = [melody, y];
    end
    plotWaveform(axOriginal, melody, fs);
    plotSpectrum(axSpectrum, melody, fs);
    sound(melody, fs);
end

% Function to play FSK modulated "Never Gonna Give You Up"
function playNGGYUFSK(axFSK, axSpectrum, bit_duration)
    fs = 44100; % Sample rate

    % Notes and durations based on the piano chords
    notes = {'C', 'D', 'F', 'D', 'A', 'A', 'G', 'C', 'D', 'F', 'D', 'G', ...
             'G', 'F', 'C', 'D', 'F', 'D', 'F', 'G', 'E', 'D', 'C', 'C', ...
             'G', 'F', 'C', 'D', 'F', 'D', 'A', 'A', 'G', 'C', 'D', 'F', ...
             'D', 'G', 'G', 'F', 'C', 'D', 'F', 'D', 'F', 'G', 'E', 'D', ...
             'C', 'C', 'G', 'F',};
    % Frequencies corresponding to the notes
    frequencies = cellfun(@getNoteFrequency, notes);

    % Generate FSK modulated melody
    t = 0:1/fs:bit_duration - 1/fs;
    y = [];
    for f = frequencies
        y = [y, sin(2*pi*f*t)];
    end

    % Plot the waveform and spectrum
    plotWaveform(axFSK, y, fs);
    plotSpectrum(axSpectrum, y, fs);

    % Play the sound
    sound(y, fs);
end
